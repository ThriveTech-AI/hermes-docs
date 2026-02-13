# API Reference

The Hermes dashboard exposes a JSON API for managing conversations, viewing stats, and maintaining the knowledge base. All endpoints are served from the Worker's `fetch()` handler.

## Authentication

Three methods, checked in order:

| Method | Parameter | Role |
|--------|-----------|------|
| API Key | `?key=KEY` or `Authorization: Bearer KEY` | Owner |
| Viewer Token | `?viewer=TOKEN` | Viewer |
| GitHub OAuth | Session cookie (automatic) | Owner/Viewer+/Viewer |

If `DASHBOARD_API_KEY` is not configured, all requests are allowed (dev mode).

## Base URL

```
https://hermes.martymcenroe.ai
```

---

## Conversations

### List Conversations

```
GET /api/conversations?state=ENGAGING&limit=50
```

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `state` | string | No | Filter by conversation state |
| `limit` | integer | No | Max results (default 50) |

**Response:** Array of conversation objects.

### Get Conversation

```
GET /api/conversations/:id
```

**Response:** Conversation object with metadata (no messages).

### Get Messages

```
GET /api/conversations/:id/messages
```

**Response:** Array of message objects ordered chronologically.

**Message object:**
```json
{
  "id": 1,
  "conversation_id": 1,
  "direction": "inbound",
  "body_text": "Hi Marty, I have a role...",
  "subject": "AI Director Position",
  "ai_model": null,
  "ai_tokens_in": null,
  "ai_tokens_out": null,
  "ai_intent": null,
  "send_success": 1,
  "created_at": "2026-02-13T12:00:00Z"
}
```

---

## Stats

### Get Dashboard Stats

```
GET /api/stats
```

**Response:**
```json
{
  "conversations": {
    "total": 42,
    "by_state": {
      "ENGAGING": 15,
      "QUALIFYING": 8,
      "STAR_REQUESTED": 5,
      "STARRED": 2,
      "GHOSTED": 12
    }
  },
  "stars": {
    "verified": 2,
    "conversion_rate": "4.8%"
  },
  "ai": {
    "enabled_conversations": 30,
    "tokens_in": 45000,
    "tokens_out": 12000,
    "estimated_neurons": 570
  },
  "activity": {
    "messages_24h": 18
  }
}
```

---

## Knowledge Base

### List Knowledge Items

```
GET /api/knowledge?category=qa_pair
```

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `category` | string | No | Filter: `career_fact`, `qa_pair`, `filled_table`, `example_conversation` |

**Response:** Array of knowledge items.

### Create Knowledge Item

```
POST /api/knowledge
Content-Type: application/json

{
  "category": "qa_pair",
  "key": "What is your rate?",
  "value": "Flexible -- depends on the role and expectations."
}
```

**Response:** Created knowledge item (status 201).

### Update Knowledge Item

```
PUT /api/knowledge/:id
Content-Type: application/json

{
  "key": "Updated question",
  "value": "Updated answer"
}
```

Both `key` and `value` are optional -- include only the fields you want to change.

**Response:** Updated knowledge item.

### Delete Knowledge Item

```
DELETE /api/knowledge/:id
```

Soft-deletes the item (sets `active = 0`). The item remains in D1 but is excluded from queries.

**Response:** `{"ok": true}`

### Import Filled Table

```
POST /api/knowledge/import-table
Content-Type: application/json

{
  "name": "AI Director Skills Matrix",
  "table": "Experience in applied Data Science | 12 yrs | Rating 7 | Last 2025\nLeading AI initiatives | 12 yrs | Rating 9 | Last 2024"
}
```

Creates a `filled_table` category entry. The table content can be any format (markdown, CSV, pipe-delimited).

**Response:** Created knowledge item (status 201).

---

## Labels

### List Labels

```
GET /api/conversations/:id/labels
```

**Response:** Array of label objects.

### Add Label (Owner)

```
POST /api/conversations/:id/labels
Content-Type: application/json

{ "label": "demo" }
```

### Remove Label (Owner)

```
DELETE /api/conversations/:id/labels
Content-Type: application/json

{ "label": "demo" }
```

---

## Ratings

### Rate a Response (Owner)

```
POST /api/ratings
Content-Type: application/json

{
  "messageId": 42,
  "conversationId": 1,
  "rating": 4,
  "note": "Good tone but star push was weak"
}
```

### Get Ratings

```
GET /api/ratings/:conversationId
```

---

## Observability

### Get Internal Metrics

```
GET /api/observability
```

Returns token usage, intent distribution, state distribution, send stats, rating distribution, messages by day, and recent AI interactions.

---

## Admin (Owner Only)

### Viewer Tokens

```
GET    /api/admin/viewer-tokens        # List all tokens
POST   /api/admin/viewer-tokens        # Generate new token (4hr TTL)
DELETE /api/admin/viewer-tokens/:id    # Revoke a token
```

### Bulk Re-init

```
POST /api/admin/reinit
Content-Type: application/json

{ "conversationIds": [1, 2, 3] }
```

Resets selected conversations to INITIAL state with AI enabled.

### Bulk Poke

```
POST /api/admin/poke
Content-Type: application/json

{ "conversationIds": [1, 2, 3] }
```

Re-init + generate AI response + send via Cloudflare binding for each conversation.

### Sweep

```
POST /api/admin/sweep
Content-Type: application/json

{
  "filter": {
    "state": "INITIAL",
    "beforeDate": "2026-02-13T16:00:00Z",
    "label": "demo"
  }
}
```

Finds all non-starred conversations matching the filter and pokes them all.

---

## Error Responses

All errors return JSON:

```json
{
  "error": "Description of what went wrong"
}
```

| Status | Meaning |
|--------|---------|
| 400 | Bad request (missing required fields) |
| 403 | Forbidden (insufficient role for this action) |
| 404 | Resource not found |
| 500 | Internal server error |

---

## Usage Examples

### Bash/curl

```bash
KEY="your-api-key"
BASE="https://hermes.martymcenroe.ai"

# List all conversations
curl -s "$BASE/api/conversations?key=$KEY" | python3 -m json.tool

# Get conversation thread
curl -s "$BASE/api/conversations/1/messages?key=$KEY" | python3 -m json.tool

# Add a Q&A pair
curl -s -X POST "$BASE/api/knowledge?key=$KEY" \
  -H "Content-Type: application/json" \
  -d '{"category":"qa_pair","key":"Do you know Kubernetes?","value":"Yes, I have been using K8s for about 5 years. Deployed production workloads on EKS and AKS."}'

# Edit an entry
curl -s -X PUT "$BASE/api/knowledge/42?key=$KEY" \
  -H "Content-Type: application/json" \
  -d '{"value":"Updated answer with more detail"}'

# Delete an entry
curl -s -X DELETE "$BASE/api/knowledge/42?key=$KEY"
```

### JavaScript/fetch

```javascript
const API_KEY = "your-api-key";
const BASE = "https://hermes.martymcenroe.ai";

// List knowledge base
const items = await fetch(`${BASE}/api/knowledge?key=${API_KEY}`).then(r => r.json());

// Create entry
await fetch(`${BASE}/api/knowledge?key=${API_KEY}`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    category: "qa_pair",
    key: "Are you available for remote?",
    value: "Absolutely, remote works great for me."
  })
});
```
