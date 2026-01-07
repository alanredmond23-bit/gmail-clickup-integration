# Tech Stack Recommendations

*Agent 5 Findings - Technology Choices*

## Frontend

### Framework: Next.js 14 App Router
**Rationale**: Already used in monorepo, SSR for fast initial load, API routes for backend

### State Management

| Type | Library | Purpose |
|------|---------|---------|
| Server State | TanStack Query | Email fetching, caching, background refresh |
| UI State | Zustand | Compose modal, selections, filters |
| Form State | React Hook Form | Compose email form |

### Offline Support: Dexie.js (IndexedDB)
**Rationale**:
- Cache emails for offline reading
- Instant UI while fetching fresh data
- Sync queue for offline actions

```typescript
// Schema
const db = new Dexie('GmailClient');
db.version(1).stores({
  emails: 'id, threadId, accountId, labelIds, date',
  threads: 'id, accountId, snippet, lastMessageDate',
  syncQueue: '++id, action, payload, timestamp'
});
```

### UI Components

| Component | Approach |
|-----------|----------|
| Email List | Virtualized list (react-window) for 1000s of emails |
| Thread View | Collapsible message cards |
| Compose Modal | Dialog with rich text (Tiptap editor) |
| Sidebar | Folder tree with badge counts |

## Backend

### Deployment: Vercel Edge Functions
**Rationale**: Already deployed on Vercel, Edge for low latency

### Database: Supabase PostgreSQL
**Rationale**: Already used in monorepo, RLS for security, real-time subscriptions

### Proposed Schema

```sql
-- Multi-account Gmail tokens
gmail_accounts (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES auth.users,
  email TEXT,
  access_token TEXT ENCRYPTED,
  refresh_token TEXT ENCRYPTED,
  token_expiry TIMESTAMPTZ,
  history_id BIGINT,
  is_primary BOOLEAN
)

-- Email metadata cache (not full content)
emails (
  id TEXT PRIMARY KEY, -- Gmail message ID
  account_id UUID REFERENCES gmail_accounts,
  thread_id TEXT,
  subject TEXT,
  snippet TEXT,
  from_email TEXT,
  from_name TEXT,
  date TIMESTAMPTZ,
  label_ids TEXT[],
  is_read BOOLEAN,
  has_attachments BOOLEAN
)

-- Conversation grouping
email_threads (
  id TEXT PRIMARY KEY,
  account_id UUID REFERENCES gmail_accounts,
  subject TEXT,
  snippet TEXT,
  message_count INT,
  last_message_date TIMESTAMPTZ,
  participants TEXT[]
)

-- ClickUp OAuth
clickup_accounts (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES auth.users,
  workspace_id TEXT,
  access_token TEXT ENCRYPTED,
  refresh_token TEXT ENCRYPTED
)

-- Bidirectional links
email_task_mappings (
  id UUID PRIMARY KEY,
  email_id TEXT REFERENCES emails,
  task_id TEXT,
  list_id TEXT,
  sync_status TEXT,
  last_synced TIMESTAMPTZ
)

-- Job tracking
sync_jobs (
  id UUID PRIMARY KEY,
  type TEXT, -- 'gmail_push', 'clickup_webhook', 'full_sync'
  status TEXT,
  payload JSONB,
  created_at TIMESTAMPTZ,
  completed_at TIMESTAMPTZ,
  error TEXT
)
```

## Real-Time Sync

### Gmail Push Notifications
1. Set up Google Cloud Pub/Sub topic
2. Create subscription → `/api/webhooks/gmail/push`
3. Call `users.watch()` on OAuth
4. Cron job: Renew every 7 days

### Polling Fallback
- If no push notification for 20 minutes → trigger poll
- During active use: 30-second polling interval

### ClickUp Webhooks
1. Register via ClickUp API
2. Handle: `taskCreated`, `taskUpdated`, `taskCommentPosted`
3. Update email labels/status accordingly

## Security

| Concern | Solution |
|---------|----------|
| Token Storage | Encrypt at rest in Supabase |
| iframe Security | CSP `frame-ancestors` whitelist |
| API Rate Limits | Queue + exponential backoff |
| CORS | Strict origin checking |

## Performance

| Optimization | Approach |
|--------------|----------|
| Large Lists | Virtualization (react-window) |
| Email Cache | IndexedDB + stale-while-revalidate |
| API Calls | Batch requests where possible |
| Initial Load | SSR skeleton + progressive hydration |
