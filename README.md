# Gmail-ClickUp Integration

A full-featured Gmail client embedded in ClickUp via iframe, with bidirectional sync between emails and ClickUp tasks. Supports multiple Gmail accounts.

## Overview

This project builds a custom Gmail client that can be embedded directly inside ClickUp's Embed View, providing:

- **Full Email Functionality**: Inbox, Sent, Drafts, Trash, All Mail, Labels
- **Compose & Reply**: Rich text editor with attachment support
- **Multi-Account Support**: Switch between multiple Gmail accounts with unified inbox
- **ClickUp Bidirectional Sync**: Convert emails to tasks, sync task updates back to email labels
- **Real-Time Updates**: Gmail Push notifications via Pub/Sub + ClickUp webhooks
- **Offline Support**: IndexedDB caching for offline access

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│  ClickUp Embed View (iframe)                                │
│  └── /embed/gmail-client (Next.js page)                     │
│      ├── AccountSwitcher (multi-account)                    │
│      ├── GmailSidebar (Inbox, Sent, Drafts, Trash, Labels)  │
│      ├── EmailList (virtualized, searchable)                │
│      ├── ThreadView (conversation view)                     │
│      └── ComposeModal (send/reply)                          │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  Vercel Edge Functions                                       │
│  ├── /api/webhooks/gmail/push    (Pub/Sub receiver)         │
│  ├── /api/webhooks/clickup       (ClickUp webhooks)         │
│  ├── /api/gmail/*                (Gmail CRUD)               │
│  └── /api/sync/*                 (Bidirectional sync)       │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  Supabase (PostgreSQL)                                       │
│  ├── gmail_accounts       (multi-account tokens)            │
│  ├── emails               (cached email metadata)           │
│  ├── email_threads        (conversation grouping)           │
│  ├── clickup_accounts     (ClickUp OAuth)                   │
│  ├── email_task_mappings  (bidirectional links)             │
│  └── sync_jobs            (job tracking)                    │
└─────────────────────────────────────────────────────────────┘
```

## Tech Stack

- **Frontend**: Next.js 14 App Router, TanStack Query, Zustand, Tailwind CSS
- **Backend**: Vercel Edge Functions, Supabase PostgreSQL
- **APIs**: Gmail API (OAuth2), ClickUp API (OAuth2)
- **Real-Time**: Google Cloud Pub/Sub, ClickUp Webhooks
- **Offline**: Dexie.js (IndexedDB)

## Project Status

**Phase**: Planning Complete ✅

See [docs/IMPLEMENTATION_PLAN.md](docs/IMPLEMENTATION_PLAN.md) for the full 10-day implementation plan.

## Key Files

| Document | Description |
|----------|-------------|
| [docs/IMPLEMENTATION_PLAN.md](docs/IMPLEMENTATION_PLAN.md) | Full implementation plan with phases, deliverables, and timeline |
| [docs/research/](docs/research/) | Agent research findings on Gmail API, ClickUp, existing codebase |
| [docs/CONVERSATION_NOTES.md](docs/CONVERSATION_NOTES.md) | Planning session summary |

## Critical Pre-Implementation Task

**MUST FIX**: The existing monorepo has `X-Frame-Options: DENY` in `vercel.json` which blocks ALL iframes. This must be updated to allow ClickUp embedding:

```json
{
  "headers": [
    {
      "source": "/embed/(.*)",
      "headers": [
        {
          "key": "Content-Security-Policy",
          "value": "frame-ancestors 'self' https://*.clickup.com"
        }
      ]
    }
  ]
}
```

## Implementation Timeline

| Phase | Days | Focus |
|-------|------|-------|
| 1. Infrastructure | 1-2 | Vercel config, Supabase schema, env vars |
| 2. Backend APIs | 3-4 | Gmail, ClickUp, Webhook routes |
| 3. Client Libraries | 4-5 | Gmail v2 client, ClickUp client, Sync orchestrator |
| 4. Frontend UI | 5-7 | Components, hooks, embed page |
| 5. Real-Time Sync | 7-8 | Push notifications, webhooks |
| 6. Testing | 9-10 | Integration tests, polish |

## Related Resources

- **Existing Codebase**: `/Users/alanredmond/monorepo/`
- **Existing Gmail Client**: `/monorepo/apps/web/lib/command-center/gmail-client.ts`
- **Supabase Migrations**: `/monorepo/03-data/supabase/migrations/`

---

*Planning session: January 2026*
