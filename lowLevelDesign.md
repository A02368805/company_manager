# 📘 Service Business Manager  
## Low-Level Technical Design (v1 – Strict Architecture Contract)

---

# ⚠️ THIS DOCUMENT IS A CONTRACT

Cursor must treat this document as the source of truth.

It must NOT:

- Invent new architecture patterns
- Move logic to random places
- Skip RLS
- Put secrets in client code
- Bypass company scoping
- Collapse all logic into frontend components
- Merge AI into non-marketing modules

All implementation must follow this design strictly.

---

# 1. Architecture Overview

## 1.1 Stack

Frontend:
- Next.js (App Router)
- TypeScript
- Server Components where appropriate
- Client Components only when necessary

Backend:
- Supabase Edge Functions (primary backend logic layer)

Database:
- Supabase Postgres

Authentication:
- Supabase Auth (email/password only in v1)

Storage:
- Supabase Storage (private buckets)

AI:
- OpenAI API
- LangGraph for orchestration

Billing:
- Stripe

Hosting:
- Vercel (frontend)
- Supabase (DB/Auth/Storage/Functions)

---

# 2. Environment Variables (REQUIRED STRUCTURE)

These must exist:

Supabase:
- NEXT_PUBLIC_SUPABASE_URL=
- NEXT_PUBLIC_SUPABASE_ANON_KEY=
- SUPABASE_SERVICE_ROLE_KEY=  (SERVER ONLY)

OpenAI:
- OPENAI_API_KEY=  (SERVER ONLY — Edge Function only)

Stripe:
- STRIPE_SECRET_KEY=
- STRIPE_WEBHOOK_SECRET=
- NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=

Rules:
- NEVER expose service role key in client.
- NEVER expose OpenAI key in client.
- All AI calls must occur in Edge Functions.

---

# 3. Multi-Tenancy Model (CRITICAL)

## 3.1 Core Rule

Every table except companies must contain:

- company_id (uuid, NOT NULL)

Every query must filter by company_id.

No exceptions.

---

## 3.2 User Model

users table:

- id (uuid) = auth.uid()
- company_id (uuid)
- email
- role (default: owner)

Each user belongs to exactly ONE company in v1.

---

# 4. Database Schema (STRICT)

All tables must include:

- id uuid primary key
- company_id uuid not null
- created_at timestamptz default now()
- updated_at timestamptz default now()
- created_by uuid
- updated_by uuid

---

## 4.1 companies

- id
- name
- service_type
- phone
- address
- employees_count
- years_in_business
- estimated_revenue
- referral_source
- subscription_status (trial | active | canceled)
- trial_started_at
- trial_ends_at

---

## 4.2 customers

- id
- company_id
- type (customer | prospect)
- name
- email
- phone
- address
- notes
- source

Indexes:
- (company_id, created_at DESC)
- (company_id, name)

---

## 4.3 jobs

- id
- company_id
- customer_id
- customer_name (snapshot)
- service_type
- status (lead | scheduled | completed | paid)
- scheduled_start
- scheduled_end
- address
- price
- notes
- source

Indexes:
- (company_id, scheduled_start)
- (company_id, status)
- (company_id, created_at DESC)

---

## 4.4 finance_entries

- id
- company_id
- type (revenue | expense)
- job_id
- title
- category
- amount (positive numeric only)
- entry_date
- notes

Indexes:
- (company_id, entry_date)
- (company_id, type)

---

## 4.5 marketing_assets

- id
- company_id
- channel (social_post | email | sms | flyer)
- context
- content
- status (draft)
- deleted_at

Indexes:
- (company_id, created_at DESC)
- (company_id, channel)

---

# 5. RLS (ROW LEVEL SECURITY) — MANDATORY

RLS must be enabled on:

- customers
- jobs
- finance_entries
- marketing_assets
- calendar_events
- job_photos

---

## 5.1 RLS Policy Pattern

SELECT:
Allow only where company_id = user's company_id

INSERT:
Allow only where company_id matches user's company_id

UPDATE:
Same restriction

DELETE:
Same restriction

---

## 5.2 Mapping Rule

Use:

auth.uid() → users table → company_id

Policies must reference this mapping.

---

## 5.3 DO NOT

- DO NOT rely on frontend filtering
- DO NOT disable RLS
- DO NOT use service role for user-scoped operations

---

# 6. Edge Function Requirements

All Edge Functions must:

1. Validate JWT
2. Extract user id
3. Load company_id
4. Enforce company_id filtering
5. Validate input schema
6. Return structured JSON

No business logic in frontend.

---

# 7. Marketing AI Architecture (LangGraph Required)

AI logic MUST only exist inside marketing module.

---

## 7.1 LangGraph Node Design

Node 1 — LoadCompanyContextNode
- Pull company name + service type

Node 2 — ValidateRequestNode
- Validate channel
- Validate length constraints

Node 3 — AssemblePromptInputsNode
- Build structured prompt input

Node 4 — GenerateCopyNode (ONLY AI CALL)
- Call OpenAI
- Generate formatted output

Node 5 — PostProcessNode
- Enforce SMS length
- Trim whitespace
- Safety filtering

Node 6 — PersistMarketingAssetNode
- Insert into marketing_assets

AI must not:
- Query database directly
- Perform validation
- Decide company scoping

---

# 8. Stripe Billing Architecture

Endpoints:

/functions/v1/billing/create-checkout-session
/functions/v1/billing/webhook

Rules:

- Webhook verifies signature
- Webhook updates company.subscription_status
- Never trust client payment result

---

# 9. UI Layout Contract

Shared AppShell component required.

Desktop:
- Sidebar visible permanently

Mobile:
- Sidebar hidden
- Hamburger menu toggles drawer

Tables:
- Must collapse into card layout on small screens
- No horizontal scroll

Design must look professional at all screen sizes.

---

# 10. Security Requirements

- HTTPS only
- Signed URLs for storage
- Short-lived upload URLs
- Input validation server-side
- Structured error responses
- Log user id + company id for critical actions

---

# 11. Performance

- Pagination required for lists
- Indexed queries mandatory
- Debounced search inputs
- Finance totals calculated server-side

---

# 12. Strict Separation Rules

AI only in Marketing module.

Stripe only in Billing module.

No cross-module coupling.

No direct DB access from client (except via Supabase client under RLS).

No service role key in browser.

---

# 13. Commenting Requirement

All core logic must include comments explaining:

- What it does
- Why it exists
- Security assumptions
- Company scoping logic

This project prioritizes clarity over cleverness.