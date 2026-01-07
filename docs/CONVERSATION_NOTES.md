# Planning Session Notes

*Gmail-ClickUp Integration - January 2026*

## Session Overview

This document captures the planning session for building a Gmail client embedded in ClickUp.

## Initial Problem Statement

**User Goal**: Have a full email experience (like Spark or Gmail) inside ClickUp - not just task-centric email.

**Features Requested**:
- Inbox, Outbox (Sent), Drafts, Trash
- All emails available
- Full compose/reply functionality

## Research Phase

Investigated current ClickUp email capabilities:
- ClickUp has "Email in ClickUp" but it's task-centric only
- "Email V2" feature is on the roadmap but not released
- No native full email client exists

**Decision**: Build a custom Gmail client since Gmail APIs are already available.

## Planning Approach

Used **5 parallel agents** in plan mode to research and design:

1. **Agent 1 (Gmail API)**: Researched Gmail API endpoints, OAuth, rate limits, push notifications
2. **Agent 2 (ClickUp Integration)**: Investigated Embed View, iframe requirements, ClickUp API
3. **Agent 3 (Open Source)**: Searched GitHub for existing Gmail client implementations
4. **Agent 4 (Existing Codebase)**: Analyzed monorepo structure and existing Gmail integration
5. **Agent 5 (Tech Stack)**: Recommended frontend/backend technologies

## Key Discoveries

### Critical Issue Found
The existing `vercel.json` has `X-Frame-Options: DENY` which **blocks ALL iframes**. This must be fixed before any ClickUp embedding will work.

### Existing Assets
- Gmail OAuth already exists at `/monorepo/apps/web/lib/command-center/gmail-client.ts`
- OAuth patterns at `/monorepo/apps/web/lib/gael-guard/gmail-oauth-integration.ts`
- Webhook handling patterns exist
- TanStack Query hooks already in use

## User Requirements Captured

| Requirement | User Selection |
|-------------|----------------|
| Deployment | Embedded in ClickUp (iframe) |
| Codebase | Use existing monorepo at `/Users/alanredmond/monorepo/` |
| ClickUp Sync | Full bidirectional (Emails ↔ Tasks) |
| Gmail Accounts | Multiple accounts with unified inbox |

## Architecture Decisions

1. **Embed Approach**: ClickUp Embed View with iframe
2. **Backend**: Vercel Edge Functions + Supabase
3. **Frontend State**: TanStack Query (server) + Zustand (UI)
4. **Offline**: Dexie.js for IndexedDB caching
5. **Real-Time**: Gmail Pub/Sub push + polling fallback
6. **Sync Strategy**: Last-write-wins with user override

## Implementation Plan

See [IMPLEMENTATION_PLAN.md](IMPLEMENTATION_PLAN.md) for the full 10-day plan with 6 phases:

1. Infrastructure Setup (Days 1-2)
2. Backend API Routes (Days 3-4)
3. Gmail Client Libraries (Days 4-5)
4. Frontend Components (Days 5-7)
5. Real-Time Sync (Days 7-8)
6. Testing & Polish (Days 9-10)

## Files to Create/Modify

### Must Modify
- `/monorepo/vercel.json` - Fix iframe security headers

### New Migrations (6 files)
- `gmail_accounts.sql`
- `emails.sql`
- `email_threads.sql`
- `clickup_accounts.sql`
- `email_task_mappings.sql`
- `sync_jobs.sql`

### New API Routes
- `/api/auth/gmail/*` - Gmail OAuth
- `/api/auth/clickup/*` - ClickUp OAuth
- `/api/gmail/*` - Email CRUD
- `/api/webhooks/gmail/push` - Push receiver
- `/api/webhooks/clickup` - Webhook handler
- `/api/sync/*` - Bidirectional sync

### New Components
- `/components/gmail-client/` - Full UI component library
- `/app/embed/gmail-client/` - Embeddable page route

## Success Criteria

- [ ] Gmail client loads in ClickUp Embed View
- [ ] Can view Inbox, Sent, Drafts, Trash, Labels
- [ ] Can compose and send emails
- [ ] Can switch between multiple Gmail accounts
- [ ] Can search emails with filters
- [ ] Can create ClickUp task from email (one-click)
- [ ] ClickUp task updates sync back to email labels
- [ ] Real-time notifications (< 5 second delay)
- [ ] Works offline with cached data

## Next Steps

When resuming this project:

1. **Start with Vercel config fix** - Update `vercel.json` for iframe embedding
2. **Create Supabase migrations** - Set up database schema
3. **Extend Gmail client** - Multi-account and full CRUD
4. **Build embed page** - `/app/embed/gmail-client/`
5. **Set up webhooks** - Gmail push + ClickUp webhooks

## Claude Code Notes

During planning, user asked about thinking modes:
- `ultrathink` keyword enables extended thinking
- Global thinking can be set via `/config`
- Opus 4.5 model has thinking on by default
- "Custom model" label just means full model ID was entered

---

*Session archived for future continuation*
