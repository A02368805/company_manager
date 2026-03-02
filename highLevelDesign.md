# 📘 Ergon Management 
## High-Level Product & System Specification (v1.1)

---

# 1. Product Overview

## 1.1 Vision

Ergon Management is a multi-tenant SaaS platform designed to help small service-based businesses:

- Organize their operations
- Track revenue and expenses
- Manage jobs and customers
- Schedule work
- Generate marketing content

Core promise:

> One place to organize your business and grow it.

The system prioritizes:
- Simplicity
- Speed
- Professional presentation
- Clear workflows
- Minimal cognitive load

This product is intentionally focused and avoids unnecessary complexity.

---

## 1.2 Target Users

Primary focus:
- Auto detailing businesses
- Window washing businesses

Future expansion:
- Other local service businesses (HVAC, landscaping, cleaning, etc.)

---

# 2. Product Model

## 2.1 SaaS Multi-Tenant Architecture

This application is a multi-tenant SaaS product.

- Each company has its own isolated dataset.
- Users belong to exactly one company in v1.
- Company data must be fully isolated at the database level.
- No cross-company access is allowed.

Security and isolation are core product principles.

---

## 2.2 Subscription Model

Each company has a subscription status:

- Trial
- Active
- Canceled

Trial includes full feature access for a limited time.

If subscription expires:
- Access to authenticated pages may be restricted (exact enforcement defined in Low-Level Design).
- Company data remains stored.

Stripe manages billing.

---

# 3. Technology Overview (Product-Level)

The following technologies are foundational to the product:

Frontend:
- Next.js (App Router)
- Responsive UI design

Backend:
- Supabase Edge Functions

Database:
- Supabase Postgres

Authentication:
- Supabase Auth (email/password only in v1)

Storage:
- Supabase Storage (private buckets)

Billing:
- Stripe

AI (Marketing Module Only):
- OpenAI API
- LangGraph orchestration

AI is strictly limited to the Marketing module.

No AI is used in:
- Finance calculations
- Scheduling
- Job management
- Dashboard analytics

## Future Mobile App Direction (Post-v1)

v1 is a responsive web application optimized for desktop and mobile browsers.

A future version may add a dedicated mobile application (iOS/Android).  
To support this, backend APIs must be designed as client-agnostic services:

- Core business logic and validation live in Supabase Edge Functions.
- Edge Functions expose stable JSON contracts that any client can call.
- The web UI should not be the only place rules are enforced.

---

# 4. Public Website (Unauthenticated)

## 4.1 Main Landing Page

### Purpose
Convert visitors into trial users.

### Required Elements

- Company logo
- Short slogan
- Clear description of value
- Strong “Start Free Trial – No Credit Card Required” button
- Smaller link to pricing
- Login button

### Behavior

- Trial button → onboarding flow
- Pricing → pricing page
- Login → sign in page

Design must:
- Look professional
- Feel modern
- Work on desktop and mobile

---

## 4.2 Pricing Page

### Layout

- Single pricing tier (v1)
- Feature list
- Subscribe button

### Behavior

- Initiates Stripe checkout session
- Subscription state stored on company record

---

## 4.3 Sign In Page

### Layout

- Email
- Password
- Login button

### Behavior

- Authenticates via Supabase Auth
- Redirects to Dashboard

Passwords are never stored directly in application tables.

---

## 4.4 Onboarding Page

### Required Fields

- Email
- Password
- Company name
- Service type

### Optional Fields

- Phone
- Address
- Number of employees
- Years in business
- Estimated revenue
- Referral source

### Behavior

- Creates company
- Creates owner user
- Starts trial
- Redirects to Dashboard

---

# 5. Authenticated Application Layout

All authenticated pages must use a shared layout shell.

## 5.1 Sidebar Navigation (Required)

Left sidebar contains:

- Logo
- Dashboard
- Schedule
- Jobs
- Customers
- Marketing
- Finance
- Account Settings

Active page is visually highlighted.

---

## 5.2 Responsive Design Contract

Desktop:
- Sidebar always visible.

Mobile:
- Sidebar hidden by default.
- Hamburger icon opens slide-over menu.
- No horizontal scrolling allowed.
- Tables collapse into vertical card layout.
- Forms stack vertically.

All pages must:
- Appear professional.
- Maintain consistent spacing and typography.
- Adjust cleanly to screen size changes.

---

# 6. Dashboard

## Purpose

Provide a high-level overview of business activity.

## Sections

- Today’s Schedule
- Upcoming Jobs
- New Prospects
- Finance Summary (Revenue / Expenses / Net)
- Marketing Reminders

## Behavior

- Each section links to relevant module.
- Finance summary auto-calculates.
- No AI used here.

Dashboard uses filtering logic only.

---

# 7. Jobs Module

## 7.1 Jobs List Page

### Features

- Add Job button
- Filters (Upcoming, Past, All)
- Sortable columns:
  - Customer Name
  - Address
  - Date Scheduled
  - Date Created
  - Status

### Behavior

- Clicking job opens edit view.
- Creating job with new name auto-creates customer.

---

## 7.2 Job Status Lifecycle

- Lead
- Scheduled
- Completed
- Paid

Status progression reflects real-world workflow.

---

# 8. Customers Module

## Features

- Add Customer button
- Search
- Filter (Customer / Prospect)
- Sortable columns

Customer profile includes:

- Contact info
- Notes
- Job history
- Revenue total

---

# 9. Schedule Module

## Views

- Week view
- Month view

Users can create:

- Job
- Event
- Task

Jobs created in Schedule are true Job records.

Schedule is not a separate data source.

---

# 10. Finance Module (Lightweight Tracking)

Not full accounting.

## Top Summary

- Revenue
- Expenses
- Net Income

Time filters:

- Week
- Month
- Year

## Entry Types

- Revenue
- Expense

Each entry may link to a Job.

Totals are auto-calculated.

No AI involved.

---

# 11. Marketing Module (AI Only Section)

This is the only AI-powered module.

## Content Types

- Social Post
- Email
- SMS
- Flyer

## Flow

1. User selects type.
2. Optional context provided.
3. AI generates content.
4. Content is saved to history.
5. User can copy or regenerate.

No automatic posting in v1.

AI responsibilities:
- Generate marketing copy only.

All validation and company logic occur outside AI.

---

# 12. Account Settings

Includes:

- Company info editing
- Subscription info
- Change password
- Sign out

---

# 13. MVP Scope (Strict)

Must include:

- Supabase Auth
- Multi-tenant isolation
- Dashboard
- Jobs
- Customers
- Schedule
- Finance tracking
- Marketing AI
- Stripe subscription integration

Must NOT include:

- Payroll
- Taxes
- Bank sync
- Inventory automation
- Complex permissions
- Social media integrations
- Auto-post scheduling

---

# 14. Design System Placeholder

To be defined:

- Primary color
- Secondary color
- Accent color
- Background color
- Text color
- Success / Warning / Error colors
- Typography scale
- Spacing system
- Button styles
- Card styles
- Border radius
- Shadow system

Design must communicate professionalism and trust.