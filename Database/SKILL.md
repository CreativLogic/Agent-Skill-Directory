---
name: database
description: Activate for Supabase schema design, SQL queries, data modeling, or database-related tasks
triggers:
  - database
  - supabase
  - schema
  - SQL
  - table
  - query
context: fork
---

# Database Skill

## Supabase Project
- Platform: Supabase (PostgreSQL)
- MCP: configured in `C:\Users\inked\.claude.json` under "supabase" key
- Run SQL via: Supabase dashboard → SQL Editor → paste → Run

## Schema Overview (as of 2026-04-12)

### Core Tables
| Table | Purpose |
|-------|---------|
| `leads` | All prospects — source of truth for acquisition |
| `outreach` | Every touchpoint with a lead (email, call, DM) |
| `clients` | Active/past clients — FK to leads |
| `client_services` | What each client is subscribed to |
| `content` | Content calendar and publishing records |
| `onboarding_templates` | 13 default onboarding steps (pre-populated) |
| `onboarding_steps` | Per-client step completion tracking |
| `revenue_snapshots` | Monthly MRR snapshots |

### Key Relationships
- `outreach.lead_id` → `leads.id`
- `clients.lead_id` → `leads.id` (one-to-one at conversion)
- `client_services.client_id` → `clients.id`
- `content.client_id` → `clients.id`
- `onboarding_steps.client_id` → `clients.id`
- `onboarding_steps.template_id` → `onboarding_templates.id`

### Enums
- `lead_status`: cold, contacted, interested, demo_scheduled, proposal_sent, won, lost, nurture
- `service_type`: conversational_agent, voice_agent, review_management, crm_setup, two_way_texting, online_scheduling, workflow_automation, reputation_management, social_media_management, custom_build
- `content_status`: draft, scheduled, published, archived
- `outreach_channel`: email, phone, sms, linkedin, facebook, instagram, referral, other

### Conventions
- All tables: UUID primary keys, `created_at` and `updated_at` timestamps
- `updated_at` auto-updated via `update_updated_at()` trigger
- JSONB columns (`config`, `performance`) for flexible metadata without schema changes
- Indexes on all FK columns and frequently queried status/date fields

## MCP Configuration (for direct DB access via Claude)
HTTP MCP with bearer token. Add to `C:\Users\inked\.claude.json`:
```json
"supabase": {
  "type": "http",
  "url": "https://mcp.supabase.com/v1/mcp",
  "headers": {
    "Authorization": "Bearer [token]"
  }
}
```
Note: Use `claude mcp add` CLI for simple MCPs. For MCPs requiring custom headers, edit `.claude.json` directly — CLI does not support `--header` flag reliably.

## Learned Patterns
- Always create the `update_updated_at()` trigger function before creating tables that use it.
- Revenue snapshots use `UNIQUE(snapshot_month)` — upsert with `ON CONFLICT` for monthly updates.
- Onboarding templates table pre-populated with 13 steps at schema creation time — no manual seed needed.
