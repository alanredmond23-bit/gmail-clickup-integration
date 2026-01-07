# ClickUp Integration Research

*Agent 2 Findings - ClickUp Integration Options*

## Embed View (Selected Approach)

ClickUp's **Embed View** allows embedding external web applications via iframe.

### How It Works
1. Create an "Embed" view in any ClickUp List/Folder
2. Paste URL to your web app
3. App loads in iframe within ClickUp

### Requirements
- **Security Headers**: Must allow ClickUp domains in `frame-ancestors`
- **HTTPS**: Required for embedding
- **Responsive Design**: Should work within ClickUp's variable widths

### Security Configuration

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

## ClickUp API

### Authentication
- **OAuth 2.0** flow
- Access Token + Refresh Token
- Scopes for tasks, comments, webhooks

### Key Endpoints

| Endpoint | Purpose |
|----------|---------|
| `GET /team` | Get workspaces |
| `GET /space/{id}/folder` | Get folders in space |
| `GET /folder/{id}/list` | Get lists in folder |
| `GET /list/{id}/task` | Get tasks in list |
| `POST /list/{id}/task` | Create task |
| `PUT /task/{id}` | Update task |
| `POST /task/{id}/comment` | Add comment |
| `POST /team/{id}/webhook` | Register webhook |

### Webhooks
- Register via API to receive task events
- Events: `taskCreated`, `taskUpdated`, `taskDeleted`, `taskCommentPosted`
- Webhook payload includes full task object

### Rate Limits
- **100 requests per minute per workspace**
- Implement queue + exponential backoff

## Bidirectional Sync Design

### Email → Task
1. User clicks "Create Task" button on email
2. API call: `POST /list/{id}/task`
3. Task created with:
   - Title: Email subject
   - Description: Email body (truncated)
   - Custom field: Email ID (for linking)
4. Store mapping in `email_task_mappings` table

### Task → Email
1. ClickUp webhook fires on task update
2. Handler looks up email mapping
3. Updates email labels/stars via Gmail API
4. Example: Task closed → Add "Completed" label to email

## Alternative Options (Not Selected)

### ClickUp App Marketplace
- Would require formal app submission
- More integration but more complexity
- Not suitable for custom internal tool

### Native Email Integration
- ClickUp's "Email V2" is planned but not released
- Current email is task-centric only
- Building custom gives more control
