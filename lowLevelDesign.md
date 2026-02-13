# 📘 Service Business Manager  
## Low-Level Design Draft (v1 – AI Limited to Marketing)

> Scope note: This low-level design describes internal components, data models, APIs, and behaviors for the v1 web app. AI is used only inside the Marketing module.

---

# 1. System Overview

## 1.1 Architecture (v1)
- Web App: Next.js (App Router)
- API Layer: Next.js Route Handlers (`/app/api/*`) or Supabase Edge Functions
- Database: Postgres
- Auth: (Choose one) Supabase Auth / Clerk / NextAuth
- Storage: Object storage for uploaded images (job photos, etc.)  
- AI Provider: OpenAI (Marketing generation only)
- Hosting: Vercel (frontend + API routes) + DB hosted (Supabase/Neon/etc.)

## 1.2 Key Non-Functional Requirements
- Fast UI interactions (lists, filtering, search)
- Secure authentication + authorization (company isolation)
- Reliable data integrity (jobs/customers/finance)
- Auditability of key actions (created_by, updated_by, timestamps)
- Simple + consistent UX patterns across modules

---

# 2. Core Concepts & Domain Rules

## 2.1 Multi-Tenancy (Company Isolation)
- Every record (except global config) is scoped to a `company_id`.
- Users belong to exactly one company in v1 (simplifies onboarding and permissions).
- All reads/writes must include `company_id` checks.

## 2.2 Relationship Rules
- Creating a Job with a new person can create a Customer record.
- Schedule “Job” creation must create/update a Job record (Schedule is a view + creation surface, not a separate source of truth for jobs).
- Finance entries may optionally link to a Job (recommended for revenue lines).

## 2.3 Status Rules
- Jobs have lifecycle statuses:
  - `lead` → `scheduled` → `completed` → `paid`
- Customers have types:
  - `customer` | `prospect`

---

# 3. Data Model (Tables)

> All tables include: `id (uuid)`, `company_id (uuid)`, `created_at`, `updated_at`, `created_by`, `updated_by`.

## 3.1 users
- `id (uuid)` (auth provider id or mapped)
- `company_id (uuid)`
- `email (text, unique)`
- `name (text, nullable)`
- `role (text)` default: `owner` (v1: owner only, optional staff later)

## 3.2 companies
- `id (uuid)`
- `name (text)`
- `service_type (text)` (detailer, window_washing, hvac, etc.)
- `phone (text, nullable)`
- `address (text, nullable)`
- `employees_count (int, nullable)`
- `years_in_business (int, nullable)`
- `estimated_revenue (numeric, nullable)`
- `referral_source (text, nullable)`
- `subscription_status (text)` (trial, active, canceled)
- `trial_started_at (timestamptz, nullable)`
- `trial_ends_at (timestamptz, nullable)`

## 3.3 customers
- `id (uuid)`
- `company_id (uuid)`
- `type (text)` enum: `customer` | `prospect`
- `name (text)`
- `email (text, nullable)`
- `phone (text, nullable)`
- `address (text, nullable)`
- `notes (text, nullable)`
- `source (text, nullable)` (lead source / reference)

Indexes:
- `(company_id, created_at desc)`
- `(company_id, name)`
- Optional: full-text search on name/address

## 3.4 jobs
- `id (uuid)`
- `company_id (uuid)`
- `customer_id (uuid, nullable)` (nullable to allow “lead job” before customer created)
- `customer_name (text, nullable)` (snapshot)
- `service_type (text, nullable)` (e.g., “Interior Detail”)
- `status (text)` enum: `lead` | `scheduled` | `completed` | `paid`
- `scheduled_start (timestamptz, nullable)`
- `scheduled_end (timestamptz, nullable)`
- `address (text, nullable)`
- `price (numeric, nullable)`
- `notes (text, nullable)`
- `source (text, nullable)` (where job came from)

Indexes:
- `(company_id, scheduled_start)`
- `(company_id, status)`
- `(company_id, created_at desc)`

## 3.5 job_photos
- `id (uuid)`
- `company_id (uuid)`
- `job_id (uuid)`
- `storage_path (text)`
- `caption (text, nullable)`

## 3.6 calendar_events
> In v1, Jobs appear on the calendar from the Jobs table. Calendar Events below are for non-job items.
- `id (uuid)`
- `company_id (uuid)`
- `type (text)` enum: `event` | `task`
- `title (text)`
- `start_at (timestamptz)`
- `end_at (timestamptz, nullable)`
- `location (text, nullable)`
- `notes (text, nullable)`

Indexes:
- `(company_id, start_at)`

## 3.7 finance_entries
- `id (uuid)`
- `company_id (uuid)`
- `type (text)` enum: `revenue` | `expense`
- `job_id (uuid, nullable)`
- `title (text)` (e.g., “Job Payment”, “Supplies”)
- `category (text, nullable)` (optional)
- `amount (numeric)` (positive value; type determines direction)
- `entry_date (date)` (the date it counts toward)
- `notes (text, nullable)`

Indexes:
- `(company_id, entry_date)`
- `(company_id, type)`

## 3.8 marketing_assets
- `id (uuid)`
- `company_id (uuid)`
- `channel (text)` enum: `social_post` | `email` | `sms` | `flyer`
- `context (text, nullable)` (user prompt / purpose)
- `content (text)` (generated text)
- `status (text)` enum: `draft` | `saved` (v1: just draft/history)
- `deleted_at (timestamptz, nullable)`

Indexes:
- `(company_id, created_at desc)`
- `(company_id, channel)`

---

# 4. API Design (Route Handlers / Edge Functions)

## 4.1 Auth / Company
- `POST /api/onboarding`
  - Input: onboarding fields
  - Output: company + user created, session established
- `GET /api/me`
  - Output: current user + company summary
- `PATCH /api/company`
  - Update company info

## 4.2 Jobs
- `GET /api/jobs?filter=upcoming|past|all&sort=scheduled_start|created_at&dir=asc|desc&query=...`
- `POST /api/jobs`
- `GET /api/jobs/:id`
- `PATCH /api/jobs/:id`
- `DELETE /api/jobs/:id` (optional v1)

Job Photo Upload:
- `POST /api/jobs/:id/photos` (returns signed upload URL or handles upload)
- `DELETE /api/job-photos/:id`

## 4.3 Customers
- `GET /api/customers?type=customer|prospect|all&sort=name|created_at&query=...`
- `POST /api/customers`
- `GET /api/customers/:id`
- `PATCH /api/customers/:id`

## 4.4 Schedule
- `GET /api/schedule?start=YYYY-MM-DD&end=YYYY-MM-DD`
  - Returns:
    - Jobs within date range (from `jobs.scheduled_start`)
    - Calendar events within date range (from `calendar_events`)
- `POST /api/schedule/event`
  - Creates `calendar_events` row with `type=event`
- `POST /api/schedule/task`
  - Creates `calendar_events` row with `type=task`
- `POST /api/schedule/job`
  - Creates a Job (same as `POST /api/jobs`) but from calendar UI

## 4.5 Finance
- `GET /api/finance/summary?range=week|month|year&anchor=YYYY-MM-DD`
  - Returns totals: revenue, expenses, net
- `GET /api/finance/entries?type=revenue|expense|all&start=...&end=...`
- `POST /api/finance/entries`
- `PATCH /api/finance/entries/:id`
- `DELETE /api/finance/entries/:id` (optional v1)

## 4.6 Marketing (AI only here)
- `POST /api/marketing/generate`
  - Input:
    - `channel` (social_post/email/sms/flyer)
    - `context` (optional)
    - `company_id` (derived from session)
  - Output:
    - generated `content`
    - saved `marketing_assets` record (status=draft)
- `GET /api/marketing/history?channel=...`
- `DELETE /api/marketing/assets/:id` (soft delete)

---

# 5. UI Components (Low-Level)

## 5.1 Shared Layout Components
- `AppShell`
  - Left sidebar nav
  - Top header title
  - Content area
- `SidebarNavItem` (active highlight)
- `PageHeader` (title + optional actions)
- `DataTable`
  - sortable columns
  - hover highlight
  - empty states
  - loading skeleton
- `Modal`
  - darkened backdrop
  - close (X top right)
  - Save bottom right

## 5.2 Forms & Validation
- All create/edit modals use the same structure:
  - client-side validation (required fields)
  - server-side validation (final authority)
- Common form fields:
  - `CustomerSelect` (searchable dropdown)
  - Date picker / time picker
  - Currency input for price/amount

---

# 6. Page-by-Page Behaviors (Implementation Notes)

## 6.1 Dashboard
- Fetch:
  - Jobs for next 7 days
  - Today schedule items
  - New prospects (last 30 days)
  - Finance summary (month)
  - Marketing reminders (simple rules)
- Clicking modules routes to page

## 6.2 Jobs Page
- Default filter: upcoming
- Sorting:
  - default by scheduled date ascending
- Create/edit:
  - Modal overlay
  - Save triggers POST/PATCH
- “Customer” entry:
  - searchable select existing customers
  - if user types new name and selects “Create new customer”, create customer first then job

## 6.3 Customers Page
- Default sort: created_at desc
- “Type” filter for prospect vs customer
- Customer details show job history:
  - `GET /api/jobs?customer_id=...`

## 6.4 Schedule Page
- Calendar grid view
- Source of truth:
  - Jobs come from `jobs` table
  - Events/tasks come from `calendar_events`
- Create:
  - “Job” uses same fields as Job modal
  - “Event/Task” uses simplified fields

## 6.5 Finance Page
- Range switcher changes queries
- Totals computed server-side (to avoid client mismatch)
- Revenue/expense entries allow optional `job_id`
- Adding entry re-fetches summary + list

## 6.6 Marketing Page
- Only AI-driven page
- Generate flow:
  1) User selects channel
  2) (Optional) adds context
  3) POST `/api/marketing/generate`
  4) Result shows in output box
  5) Automatically saved to history
- History:
  - list of generated assets
  - delete = soft delete

---

# 7. Security & Authorization

## 7.1 Auth
- All `/app/*` pages require authenticated session (except public site)
- API routes require authentication

## 7.2 Authorization
- Every query enforces `company_id = session.company_id`
- Reject requests where company mismatch

## 7.3 Data Protection
- Passwords handled by auth provider (never stored in app DB directly)
- Use HTTPS in production
- Store uploaded files in private buckets with signed URLs

---

# 8. Error Handling & Observability

## 8.1 Error States (UI)
- Empty lists show friendly empty state + CTA (“Add your first job”)
- Form submission errors show field-level messages
- Network errors show toast + retry

## 8.2 Logging (Server)
- Log:
  - API request id
  - user id
  - company id
  - operation name (create job, edit customer, generate marketing)
- Avoid logging sensitive data (passwords, tokens)

---

# 9. Performance Considerations

- Pagination for large lists (jobs/customers/finance entries)
- Debounced search input
- Indexed queries (company_id + sort field)
- Cache common dashboard queries (short TTL) if needed

---

# 10. Future Enhancements (Post-v1)
- Roles/permissions (owner/manager/staff)
- Invoices + payment tracking
- Review reminders automation (non-AI, scheduled messages)
- Inventory tracking
- Integrations (Google Calendar, Stripe, etc.)
- Auto-posting / scheduling (later)
