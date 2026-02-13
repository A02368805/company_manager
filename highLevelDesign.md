# 📘 Service Business Manager  
## High-Level Design Document (v1 – Simplified, AI Limited to Marketing)

---

# 1. Product Overview

## Vision

A simple, organized platform that helps small service businesses manage:

- Jobs
- Customers
- Scheduling
- Light financial tracking
- Marketing content creation

Core promise:

> One place to organize your business and grow it.

Target Users:
- Auto detailing
- Window washing
- Expandable to other service businesses later

---

# 2. System Architecture Overview

### Application Structure

- Public website (marketing site)
- Authenticated application
- Modular page-based layout

Core principles:

- Clean and simple
- Fast workflows
- No unnecessary complexity
- Easy to learn

---

# 3. Public Website (Unauthenticated)

## 3.1 Main Landing Page

### Purpose
Convert visitors into free trial users.

### Layout

- Company logo at top
- Short slogan underneath
- Clear description of what the software does
- Strong “Start Free Trial – No Credit Card Required” button
- Smaller link to pricing
- Login button for existing users
- Professional, minimal design

### Behavior

- No login required
- Free trial button → onboarding
- Pricing link → pricing page
- Login → sign in page

---

## 3.2 Pricing Page

### Layout

- Single pricing option (v1)
- Feature list
- Subscribe button

### Behavior

- Begins subscription process

---

## 3.3 Sign In Page

### Layout

- Email
- Password
- Login button

### Behavior

- Authenticates user
- Redirects to Dashboard

---

## 3.4 Onboarding Page

### Required Fields

- Email (username)
- Password (securely stored)
- Company name
- Service type (dropdown)

### Optional Fields

- Phone
- Address
- Number of employees
- Years in business
- Estimated revenue
- How they heard about us

### Behavior

- Creates company
- Creates owner user
- Redirects to dashboard

---

# 4. Authenticated Application Layout

All internal pages share:

### Left Sidebar Navigation

- Logo
- Dashboard
- Schedule
- Jobs
- Customers
- Marketing
- Finance
- Account Settings

Active page is highlighted.

---

# 5. Dashboard

## Purpose

Provide a clear overview of the business.

## Layout

Modular sections:

- Today’s Schedule
- Upcoming Jobs
- New Prospects
- Finance Summary (Revenue / Expenses / Net)
- Marketing Reminders

## Behavior

- Clicking a section navigates to the relevant page
- Finance summary auto-calculates totals
- Marketing box may show reminders (e.g., “Follow up with inactive customers”)

No AI logic here — just smart filtering and reminders.

---

# 6. Jobs Module

## Jobs List Page

### Layout

- Header: “Jobs”
- + Add Job button
- Dropdown filter:
  - Upcoming
  - Past
  - All
- Sortable columns:
  - Customer Name
  - Address
  - Date Scheduled
  - Date Created
  - Status

### Behavior

- + opens modal
- Clicking job opens edit view
- Creating job with new name auto-creates customer

---

## Create / Edit Job Modal

Fields:

- Customer (searchable dropdown)
- Address
- Date scheduled
- Date created (auto)
- Price
- Notes
- Photos
- Lead source

Status options:
- Lead
- Scheduled
- Completed
- Paid

---

# 7. Customers Module

## Customers List Page

### Layout

- + Add Customer
- Search bar
- Filters:
  - Customer
  - Prospect
- Sortable columns:
  - Name
  - Address
  - Phone
  - Email
  - Type

Default sort: recently created.

### Behavior

- Clicking customer opens profile
- Profile shows:
  - Contact info
  - Notes
  - Job history
  - Revenue from that customer

---

# 8. Schedule Module

## Layout

- Week and Month view
- Click to create:
  - Job
  - Event
  - Task
- Optional color coding

## Behavior

- Creating a Job links to Jobs page
- Events are standalone
- Tasks link to task system
- Updates automatically reflect across system

---

# 9. Finance Module (Lightweight Tracking)

Not full accounting.

## Layout

Top Summary Boxes:

- Revenue
- Net Income (largest)
- Expenses

Time filter:
- Week
- Month
- Year

Below:
Two columns:

Revenue list  
Expense list  

Each row:
- Amount
- Date
- Linked job (optional)

## Behavior

- + Add Revenue
- + Add Expense
- Totals auto-calculate
- Net = Revenue – Expenses

---

# 10. Marketing Module (Only AI-Driven Section)

This is the only area using AI.

## Layout

Top section:
Buttons to generate:

- Social Post
- Email
- SMS
- Flyer

Optional context box:
- “What is this for?” (e.g., review reminder, seasonal promotion)

Output display box:
- Generated content
- Copy button
- Regenerate button

Lower section:
- History of created content
- Delete option

## Behavior

- User selects type
- User optionally adds context
- AI generates content
- Content can be copied and used externally
- No auto-posting in v1

---

# 11. Account Settings

## Layout

- Company info (editable)
- Subscription info
- Change password
- Sign out

## Behavior

- Updates stored company data
- Updates subscription

---

# 12. MVP Scope

Must include:

- Authentication
- Company onboarding
- Dashboard
- Jobs
- Customers
- Schedule
- Finance tracking
- Marketing content generator

Not included in v1:

- Payroll
- Taxes
- Bank sync
- Inventory automation
- Complex permissions
- Social media integrations
