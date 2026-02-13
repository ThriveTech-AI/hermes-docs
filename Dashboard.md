# Dashboard

The Hermes dashboard is a single-page web app served inline from the Worker. No build step, no separate hosting — just vanilla JS + fetch.

## Access

The dashboard supports three authentication methods and three access roles.

### Authentication Methods

| Method | How | Role Assigned |
|--------|-----|---------------|
| API Key | `?key=...` or `Authorization: Bearer ...` | Owner |
| GitHub OAuth | Sign in with GitHub button | Owner (repo owner), Viewer+ (org member), Viewer (anyone else) |
| Viewer Token | `?viewer=TOKEN` (admin-generated, 4hr TTL) | Viewer |

### Roles

| Role | Conversations | Knowledge Base | Admin | Labels | Ratings |
|------|--------------|----------------|-------|--------|---------|
| **Owner** | Full access | CRUD | Full access | Add/remove | Rate responses |
| **Viewer+** | Read all | Read only | No access | Read only | No |
| **Viewer** | Demo-labeled only (anonymized) | No access | No access | No | No |

## Tabs

### Conversations

Gmail-style conversation list with:
- **Checkboxes** — select individual conversations or use select-all
- **Filter bar** — filter by state and/or label
- **Bulk actions** — Re-init or Poke selected conversations
- **Click to open** — see full message thread with metadata

**Conversation detail** shows:
- Sender, subject, state, channel, AI enabled status, star status
- Label chips (owner can add/remove)
- Full message thread (inbound + outbound)
- Emoji rating bar on outbound AI messages (owner only, 1-5 scale with notes)

### Knowledge Base

CRUD for the four knowledge categories:
- **Q&A Pairs** — recruiter question + approved answer
- **Career Facts** — skills, experience, certifications
- **Filled Tables** — pre-filled skills matrices and forms
- **Example Conversations** — few-shot patterns for the AI

Owner can add, edit, and delete entries. Viewer+ sees read-only. Viewer has no access.

### Stats

Overview cards:
- Total conversations, stars verified, star conversion rate
- AI-enabled conversations, token usage, estimated neurons
- Messages in last 24 hours

Conversations by state breakdown.

### Observability

Internal metrics from D1:
- **Token usage** — total in/out, neuron estimate, AI vs template message count
- **Send stats** — success/failure rate
- **Intent distribution** — what recruiters are asking for
- **State distribution** — where conversations are in the pipeline
- **Rating distribution** — feedback on AI responses
- **Messages by day** — 14-day activity chart
- **Recent AI interactions** — last 20 AI-generated responses with metadata

Click any recent AI interaction to jump to that conversation.

### Admin (Owner Only)

**Viewer Tokens:**
- Generate time-limited links (4-hour TTL) for sharing during interviews
- View all tokens with status (Active/Expired/Revoked)
- Revoke active tokens

**Sweep (Bulk Poke):**
- Filter by state, date, and label
- Run sweep to re-init + send fresh AI emails to all matching non-starred conversations
- Results show per-conversation success/failure with response preview
- Uses Cloudflare send_email binding (free, unlimited — no Resend quota hit)

## Viewer Anonymization

When a viewer accesses the dashboard, all PII is masked at runtime:
- Email addresses become `Recruiter #N`
- Phone numbers become `<phone-number>`
- External URLs become `<company-url>` (github.com and martymcenroe.ai are preserved)
- Sender names are stripped

Viewers can only open conversations with the `demo` label. No data is duplicated — masking happens on the fly in the API response.

## Poke Engine

The poke engine is how you re-engage recruiters from the dashboard.

**Re-init:** Reset conversation state to INITIAL, enable AI. No email sent.

**Poke:** Re-init + generate fresh AI response + send via Cloudflare send_email binding. Treats it as if the recruiter's original email just arrived again.

**Sweep:** Apply poke to all conversations matching a filter (state, date, label).

All pokes go from `recruit@martymcenroe.ai`. The Cloudflare send_email binding is free and unlimited (no Resend rate limits).
