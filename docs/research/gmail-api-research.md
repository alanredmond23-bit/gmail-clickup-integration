# Gmail API Research

*Agent 1 Findings - Gmail API Capabilities*

## Core Capabilities

### Authentication
- **OAuth 2.0** with offline access for refresh tokens
- Scopes needed:
  - `gmail.readonly` - Read emails
  - `gmail.send` - Send emails
  - `gmail.modify` - Labels, archive, delete
  - `gmail.compose` - Create drafts

### Key Endpoints

| Endpoint | Purpose |
|----------|---------|
| `users.messages.list` | List emails with pagination |
| `users.messages.get` | Get full email content |
| `users.messages.send` | Send new email |
| `users.threads.list` | List conversation threads |
| `users.threads.get` | Get full thread with all messages |
| `users.labels.list` | Get all labels (Inbox, Sent, custom) |
| `users.drafts.*` | Draft CRUD operations |
| `users.history.list` | Incremental sync via historyId |
| `users.watch` | Set up push notifications |

### Rate Limits
- **250 quota units per second per user**
- `messages.list` = 5 units
- `messages.get` = 5 units
- `messages.send` = 100 units
- Batch requests reduce overhead

### Push Notifications (Pub/Sub)
- Real-time notifications via Google Cloud Pub/Sub
- Call `users.watch()` to start notifications
- **7-day expiration** - must renew via cron job
- Notification payload includes `historyId` for incremental sync

## Incremental Sync Strategy

```
1. Initial sync: Fetch all messages (paginated)
2. Store latest historyId
3. On push notification:
   - Extract new historyId
   - Call history.list(startHistoryId=stored)
   - Get only changed messages
4. Renew watch every 7 days
```

## Multi-Account Support

- Each account needs separate OAuth tokens
- Store in database: `gmail_accounts` table
- Token refresh handled automatically by googleapis library
- User can switch accounts via UI

## Key Libraries

- **googleapis** (Node.js) - Official Google API client
- **google-auth-library** - OAuth2 handling with token refresh

## Considerations

1. **Token Storage**: Encrypt refresh tokens in Supabase
2. **Quota Management**: Implement caching to reduce API calls
3. **Error Handling**: Handle 429 (rate limit) with exponential backoff
4. **Push Reliability**: Fallback to polling if no push for 20 minutes
