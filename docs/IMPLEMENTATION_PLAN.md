# Gmail Client for ClickUp - Implementation Plan

## Overview
Build a full-featured Gmail client (inbox, sent, drafts, trash, compose) embedded in ClickUp via iframe, with bidirectional sync between emails and ClickUp tasks. Supports multiple Gmail accounts.

---

## User Requirements
- **Deployment**: Embedded in ClickUp (iframe)
- **Codebase**: Use existing monorepo at `/Users/alanredmond/monorepo/`
- **ClickUp Sync**: Full bidirectional (Emails ↔ Tasks)
- **Accounts**: Multiple Gmail accounts with unified inbox

---

## Architecture Summary

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
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  External APIs                                               │
│  ├── Gmail API (OAuth2 + Push via Pub/Sub)                  │
│  └── ClickUp API (OAuth2 + Webhooks)                        │
└─────────────────────────────────────────────────────────────┘
```

---

## Phase 1: Infrastructure Setup (Day 1-2)

### 1.1 Fix Vercel Security Headers (CRITICAL)
**File**: `/Users/alanredmond/monorepo/vercel.json`

**Problem**: Current `X-Frame-Options: DENY` blocks ALL iframes.

**Change**:
```json
{
  "headers": [
    {
      "source": "/embed/(.*)",
      "headers": [
        {
          "key": "Content-Security-Policy",
          "value": "frame-ancestors 'self' https://*.clickup.com"
        },
        {
          "key": "X-Frame-Options",
          "value": "ALLOW-FROM https://app.clickup.com"
        }
      ]
    }
  ]
}
```

### 1.2 Supabase Migrations
**Location**: `/Users/alanredmond/monorepo/03-data/supabase/migrations/`

Create 6 migration files:
1. `20260106000001_gmail_accounts.sql` - Multi-account Gmail tokens
2. `20260106000002_emails.sql` - Email metadata cache with full-text search
3. `20260106000003_email_threads.sql` - Conversation grouping
4. `20260106000004_clickup_accounts.sql` - ClickUp OAuth tokens
5. `20260106000005_email_task_mappings.sql` - Bidirectional links
6. `20260106000006_sync_jobs.sql` - Sync tracking + operation queue

### 1.3 Environment Variables
Add to Vercel:
```
GOOGLE_CLOUD_PROJECT_ID=
GOOGLE_PUBSUB_TOPIC=gmail-push
CLICKUP_CLIENT_ID=
CLICKUP_CLIENT_SECRET=
```

---

## Phase 2: Backend API Routes (Day 3-4)

### 2.1 Gmail Routes
**Location**: `/Users/alanredmond/monorepo/apps/web/app/api/gmail/`

| Route | Method | Purpose |
|-------|--------|---------|
| `/api/auth/gmail/connect` | POST | Initiate OAuth |
| `/api/auth/gmail/callback` | GET | Handle OAuth callback |
| `/api/gmail/accounts` | GET | List connected accounts |
| `/api/gmail/emails` | GET | List emails (paginated) |
| `/api/gmail/threads/[id]` | GET | Get thread with messages |
| `/api/gmail/send` | POST | Send email |
| `/api/gmail/search` | GET | Full-text search |

### 2.2 ClickUp Routes
**Location**: `/Users/alanredmond/monorepo/apps/web/app/api/clickup/`

| Route | Method | Purpose |
|-------|--------|---------|
| `/api/auth/clickup/connect` | POST | Initiate OAuth |
| `/api/auth/clickup/callback` | GET | Handle callback |
| `/api/clickup/tasks/from-email` | POST | Create task from email |
| `/api/sync/email-to-clickup` | POST | Sync email to task |
| `/api/sync/clickup-to-email` | POST | Sync task to email |

### 2.3 Webhook Handlers
| Route | Purpose |
|-------|---------|
| `/api/webhooks/gmail/push` | Receive Gmail Pub/Sub notifications |
| `/api/webhooks/clickup` | Receive ClickUp task events |

---

## Phase 3: Gmail Client Libraries (Day 4-5)

### 3.1 Extend Existing Gmail Client
**Base**: `/Users/alanredmond/monorepo/apps/web/lib/command-center/gmail-client.ts`

**New file**: `/apps/web/lib/gmail/gmail-client-v2.ts`

Features to add:
- Multi-account token management
- Incremental sync via `historyId`
- Thread fetching with all messages
- Full CRUD (send, archive, delete, label)
- Search with Gmail query syntax

### 3.2 ClickUp Client
**New file**: `/apps/web/lib/clickup/clickup-client.ts`

Features:
- Workspace/Space/List navigation
- Task CRUD
- Comment sync (for email replies)
- Webhook management

### 3.3 Sync Orchestrator
**New file**: `/apps/web/lib/sync/sync-orchestrator.ts`

Features:
- Handle Gmail push notifications
- Handle ClickUp webhooks
- Conflict resolution (last-write-wins)
- Operation queue with retry

---

## Phase 4: Frontend Components (Day 5-7)

### 4.1 New Route for Embed
**Location**: `/Users/alanredmond/monorepo/apps/web/app/embed/gmail-client/`

```
app/embed/gmail-client/
├── layout.tsx          # Iframe-optimized (no header/sidebar)
├── page.tsx            # Main entry
└── loading.tsx         # Loading skeleton
```

### 4.2 Component Structure
**Location**: `/apps/web/components/gmail-client/`

```
components/gmail-client/
├── GmailClient.tsx              # Root layout
├── sidebar/
│   ├── GmailSidebar.tsx         # Folder navigation
│   ├── AccountSwitcher.tsx      # Multi-account dropdown
│   └── LabelList.tsx            # Custom labels
├── email-list/
│   ├── EmailListContainer.tsx   # Virtualized list
│   ├── EmailListItem.tsx        # Email row
│   └── BulkActionBar.tsx        # Multi-select actions
├── thread-view/
│   ├── ThreadContainer.tsx      # Full thread
│   ├── MessageCard.tsx          # Single message
│   └── QuickReplyBox.tsx        # Inline reply
├── compose/
│   ├── ComposeModal.tsx         # Compose dialog
│   └── RichTextEditor.tsx       # Email body editor
├── search/
│   ├── SearchBar.tsx            # Search input
│   └── SearchFilters.tsx        # Advanced filters
└── hooks/
    ├── useGmailAccounts.ts      # Multi-account state
    ├── useEmailList.ts          # TanStack Query for emails
    ├── useEmailThread.ts        # Thread fetching
    └── useOfflineSync.ts        # IndexedDB caching
```

### 4.3 State Management
- **Server State**: TanStack Query (already in monorepo)
- **UI State**: Zustand store for selections, compose state
- **Offline Cache**: Dexie.js (IndexedDB) for email caching

---

## Phase 5: Real-Time Sync (Day 7-8)

### 5.1 Gmail Push Notifications
1. Set up Google Cloud Pub/Sub topic
2. Create subscription pointing to `/api/webhooks/gmail/push`
3. Implement `users.watch()` to start notifications
4. Renew watch every 7 days (cron job)

### 5.2 Polling Fallback
- If no push for 20 minutes → trigger `history.list()` poll
- Poll interval: 30 seconds during active use

### 5.3 ClickUp Webhooks
1. Register webhook via ClickUp API
2. Handle events: taskCreated, taskUpdated, taskCommentPosted
3. Sync task comments → email replies

---

## Phase 6: Testing & Polish (Day 9-10)

### 6.1 Testing
- Unit tests for Gmail/ClickUp clients
- Integration tests for sync flow
- E2E test: embed in ClickUp, send email, verify task created

### 6.2 Polish
- Loading skeletons
- Error boundaries
- Rate limit handling UI
- Offline indicator

---

## Critical Files to Modify

| File | Action | Purpose |
|------|--------|---------|
| `/monorepo/vercel.json` | MODIFY | Enable iframe embedding for ClickUp |
| `/monorepo/03-data/supabase/migrations/` | ADD 6 files | New database schema |
| `/monorepo/apps/web/lib/command-center/gmail-client.ts` | EXTEND | Multi-account, threads |
| `/monorepo/apps/web/app/api/` | ADD routes | Gmail, ClickUp, Webhooks, Sync |
| `/monorepo/apps/web/app/embed/gmail-client/` | CREATE | Embeddable page |
| `/monorepo/apps/web/components/gmail-client/` | CREATE | All UI components |
| `/monorepo/apps/web/middleware.ts` | MODIFY | Add embed path handling |

---

## Existing Patterns to Follow

| Pattern | Location | Use For |
|---------|----------|---------|
| OAuth flow | `/lib/gael-guard/gmail-oauth-integration.ts` | Extend for multi-account |
| API hooks | `/components/command-center/hooks/useCommandCenterData.ts` | TanStack Query patterns |
| Webhook handling | `/api/webhooks/twilio/sms/route.ts` | Signature validation |
| Rate limiting | `/lib/gael-guard/api-framework.ts` | Exponential backoff |
| UI components | `/components/command-center/communication/` | Email list styling |

---

## Estimated Timeline

| Phase | Days | Deliverable |
|-------|------|-------------|
| Infrastructure | 1-2 | Vercel config, Supabase schema, env vars |
| Backend APIs | 3-4 | All API routes for Gmail, ClickUp, Webhooks |
| Client Libraries | 4-5 | Gmail v2 client, ClickUp client, Sync orchestrator |
| Frontend UI | 5-7 | All components, hooks, embed page |
| Real-Time Sync | 7-8 | Push notifications, webhooks, conflict resolution |
| Testing | 9-10 | Tests, polish, deployment |

**Total: ~10 days**

---

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
- [ ] Rate limits handled gracefully

---

## Risk Mitigation

| Risk | Mitigation |
|------|------------|
| ClickUp blocks iframe | Test early; use CSP frame-ancestors |
| Gmail API rate limits | Cache aggressively; use push not poll |
| ClickUp API limits | Queue operations; exponential backoff |
| OAuth token expiry | Auto-refresh; graceful re-auth flow |
| Data conflicts | Last-write-wins with user override option |
