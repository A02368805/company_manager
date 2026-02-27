# Docs (Read First)

Cursor (and developers) must read these before implementing anything:

1) `service_business_manager_high_level_design_v1.md`
2) `service_business_manager_low_level_design_v1.md`

If there is any conflict, the Low-Level Design wins.

Key non-negotiables:
- Multi-tenant SaaS (company isolation)
- RLS mandatory (never rely on frontend filtering)
- Supabase Auth only
- Supabase Edge Functions for backend logic
- AI ONLY in Marketing via LangGraph
- Stripe isolated to Billing endpoints
- Responsive professional UI (desktop sidebar, mobile hamburger drawer)
- Lots of comments in core logic