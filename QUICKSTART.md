# Quickstart: Continue Gmail-ClickUp Integration

When you're ready to resume this project, follow these steps in order.

---

## Pre-Flight Checklist

Before coding, ensure you have:

- [ ] Access to Google Cloud Console (for Pub/Sub setup)
- [ ] ClickUp admin access (for OAuth app registration)
- [ ] Supabase project credentials
- [ ] Monorepo at `/Users/alanredmond/monorepo/` accessible

---

## Phase 1: Fix Critical Blocker (30 min)

### Step 1.1: Update Vercel Security Headers

**This is blocking everything** - current config prevents ClickUp iframe embedding.

```bash
# Open the file
code /Users/alanredmond/monorepo/vercel.json
```

**Add this configuration** (or modify existing headers):

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

### Step 1.2: Deploy & Test Iframe

```bash
# Deploy to Vercel
cd /Users/alanredmond/monorepo
vercel --prod

# Test: Create a simple test page
# /app/embed/test/page.tsx with just "Hello from embed!"
# Then add as ClickUp Embed View to verify it loads
```

---

## Phase 2: Database Schema (1-2 hours)

### Step 2.1: Create Migration Files

```bash
cd /Users/alanredmond/monorepo/03-data/supabase/migrations
```

Create these 6 files in order:

| File | Purpose |
|------|---------|
| `20260107000001_gmail_accounts.sql` | Multi-account Gmail tokens |
| `20260107000002_emails.sql` | Email metadata cache |
| `20260107000003_email_threads.sql` | Conversation grouping |
| `20260107000004_clickup_accounts.sql` | ClickUp OAuth tokens |
| `20260107000005_email_task_mappings.sql` | Email ↔ Task links |
| `20260107000006_sync_jobs.sql` | Sync job tracking |

### Step 2.2: Run Migrations

```bash
supabase db push
# or
supabase migration up
```

---

## Phase 3: Extend Gmail Client (2-3 hours)

### Step 3.1: Create New Gmail Client

**Base on existing**: `/monorepo/apps/web/lib/command-center/gmail-client.ts`

**New file**: `/monorepo/apps/web/lib/gmail/gmail-client-v2.ts`

**Features to add**:
- [ ] Multi-account token management (store/retrieve from Supabase)
- [ ] Incremental sync via `historyId`
- [ ] Thread fetching with all messages
- [ ] Full CRUD: send, archive, delete, label
- [ ] Search with Gmail query syntax

### Step 3.2: Create ClickUp Client

**New file**: `/monorepo/apps/web/lib/clickup/clickup-client.ts`

**Features**:
- [ ] OAuth flow
- [ ] Workspace/Space/List navigation
- [ ] Task CRUD
- [ ] Comment sync

---

## Phase 4: Build Embed Page (2-3 hours)

### Step 4.1: Create Route Structure

```bash
mkdir -p /Users/alanredmond/monorepo/apps/web/app/embed/gmail-client
```

Create these files:
- `layout.tsx` - Minimal layout (no main nav/sidebar)
- `page.tsx` - Main Gmail client entry
- `loading.tsx` - Loading skeleton

### Step 4.2: Create Components

```bash
mkdir -p /Users/alanredmond/monorepo/apps/web/components/gmail-client
```

Start with these core components:
1. `GmailClient.tsx` - Root layout
2. `GmailSidebar.tsx` - Folder navigation
3. `EmailList.tsx` - Message list (virtualized)
4. `ThreadView.tsx` - Conversation view
5. `ComposeModal.tsx` - Send/reply dialog

---

## Phase 5: API Routes (2-3 hours)

### Step 5.1: Gmail API Routes

```
/app/api/auth/gmail/connect/route.ts    - Start OAuth
/app/api/auth/gmail/callback/route.ts   - Handle callback
/app/api/gmail/accounts/route.ts        - List accounts
/app/api/gmail/emails/route.ts          - List/search emails
/app/api/gmail/threads/[id]/route.ts    - Get thread
/app/api/gmail/send/route.ts            - Send email
```

### Step 5.2: ClickUp API Routes

```
/app/api/auth/clickup/connect/route.ts  - Start OAuth
/app/api/auth/clickup/callback/route.ts - Handle callback
/app/api/clickup/tasks/from-email/route.ts - Create task from email
```

### Step 5.3: Webhook Routes

```
/app/api/webhooks/gmail/push/route.ts   - Gmail Pub/Sub receiver
/app/api/webhooks/clickup/route.ts      - ClickUp events
```

---

## Phase 6: Real-Time Sync (2-3 hours)

### Step 6.1: Gmail Push Setup

1. Go to Google Cloud Console
2. Create Pub/Sub topic: `gmail-push`
3. Create subscription → `https://your-domain.vercel.app/api/webhooks/gmail/push`
4. Grant Gmail push permissions to topic

### Step 6.2: ClickUp Webhook Setup

```typescript
// Register via API
POST https://api.clickup.com/api/v2/team/{team_id}/webhook
{
  "endpoint": "https://your-domain.vercel.app/api/webhooks/clickup",
  "events": ["taskCreated", "taskUpdated", "taskCommentPosted"]
}
```

---

## Quick Commands Reference

```bash
# Navigate to monorepo
cd /Users/alanredmond/monorepo

# Start dev server
pnpm dev

# Run Supabase locally
supabase start

# Push database changes
supabase db push

# Deploy to Vercel
vercel --prod

# Check this planning repo
cd /Users/alanredmond/githubrepos/gmail-clickup-integration
```

---

## Key Files Reference

| What | Where |
|------|-------|
| Implementation Plan | `docs/IMPLEMENTATION_PLAN.md` |
| Gmail API Research | `docs/research/gmail-api-research.md` |
| ClickUp Research | `docs/research/clickup-integration-research.md` |
| Existing Gmail Client | `/monorepo/apps/web/lib/command-center/gmail-client.ts` |
| Existing OAuth Pattern | `/monorepo/apps/web/lib/gael-guard/gmail-oauth-integration.ts` |
| Vercel Config (FIX!) | `/monorepo/vercel.json` |

---

## Estimated Time to MVP

| Phase | Time |
|-------|------|
| 1. Fix iframe headers | 30 min |
| 2. Database schema | 1-2 hours |
| 3. Gmail client v2 | 2-3 hours |
| 4. Embed page + components | 2-3 hours |
| 5. API routes | 2-3 hours |
| 6. Real-time sync | 2-3 hours |
| **Total** | **~10-15 hours** |

---

*Last updated: January 2026*
