# Existing Codebase Analysis

*Agent 4 Findings - Monorepo at `/Users/alanredmond/monorepo/`*

## Monorepo Structure

```
monorepo/
├── apps/
│   └── web/                    # Main Next.js application
│       ├── app/                # App Router pages
│       │   ├── api/            # API routes
│       │   └── embed/          # (To be created)
│       ├── components/         # React components
│       │   └── command-center/ # Existing command center UI
│       └── lib/                # Shared libraries
│           ├── command-center/ # Gmail client exists here!
│           └── gael-guard/     # OAuth integration
├── 03-data/
│   └── supabase/
│       └── migrations/         # Database migrations
└── vercel.json                 # Deployment config (NEEDS UPDATE!)
```

## Critical Existing Files

### 1. Gmail Client (TO EXTEND)
**Path**: `/apps/web/lib/command-center/gmail-client.ts`

```typescript
// Current capabilities:
- Single account OAuth
- Fetch last 50 messages from 7 days
- AI impact classification
- Basic email display

// Needs extension for:
- Multi-account token management
- Full CRUD (send, archive, delete)
- Thread fetching
- Incremental sync via historyId
```

### 2. Gmail OAuth Integration
**Path**: `/apps/web/lib/gael-guard/gmail-oauth-integration.ts`

```typescript
// Uses googleapis library
- OAuth2 flow implementation
- Token refresh handling
- Proper error handling patterns

// Pattern to follow for:
- ClickUp OAuth
- Multi-account Gmail OAuth
```

### 3. Webhook Handler Pattern
**Path**: `/apps/web/app/api/webhooks/twilio/sms/route.ts`

```typescript
// Shows:
- Signature validation
- Async processing
- Error handling
- Logging patterns
```

### 4. Rate Limiting Pattern
**Path**: `/apps/web/lib/gael-guard/api-framework.ts`

```typescript
// Implements:
- Exponential backoff
- Retry logic
- Rate limit tracking
```

### 5. TanStack Query Hooks
**Path**: `/components/command-center/hooks/useCommandCenterData.ts`

```typescript
// Pattern for:
- Server state management
- Caching strategy
- Optimistic updates
```

## CRITICAL: Vercel Configuration

**Path**: `/vercel.json`

**Current Problem**:
```json
{
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        { "key": "X-Frame-Options", "value": "DENY" }
      ]
    }
  ]
}
```

**This BLOCKS ALL iframes!** Must be updated to allow ClickUp embedding.

## Supabase Migrations Location

**Path**: `/03-data/supabase/migrations/`

New migrations to create:
1. `20260106000001_gmail_accounts.sql`
2. `20260106000002_emails.sql`
3. `20260106000003_email_threads.sql`
4. `20260106000004_clickup_accounts.sql`
5. `20260106000005_email_task_mappings.sql`
6. `20260106000006_sync_jobs.sql`

## Existing Patterns to Reuse

| Pattern | Location | Use For |
|---------|----------|---------|
| OAuth flow | `/lib/gael-guard/gmail-oauth-integration.ts` | Extend for multi-account |
| API hooks | `/components/command-center/hooks/useCommandCenterData.ts` | TanStack Query patterns |
| Webhook handling | `/api/webhooks/twilio/sms/route.ts` | Signature validation |
| Rate limiting | `/lib/gael-guard/api-framework.ts` | Exponential backoff |
| UI components | `/components/command-center/communication/` | Email list styling |
