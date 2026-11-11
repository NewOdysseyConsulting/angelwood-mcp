---
name: angelwood-api
description: Read and create records in Angelwood (angelwood.io), the client portal and client operations platform for agencies, through its REST API with an API key. Use when the user asks to list or find clients, projects, tasks or invoices in Angelwood, check which invoices are overdue or paid, add a client, or add a task to a project, and no Angelwood MCP server is connected.
---

# Angelwood API

Angelwood is a client portal and client operations platform for agencies, consultancies and other service businesses.
This skill calls its REST API directly. It can read clients, projects, tasks and invoices, and create clients and
tasks. It cannot update or delete anything.

If the `angelwood` MCP server is connected, prefer the `angelwood` skill for lookups; use this one when the user wants
to create a client or a task, or works with an API key.

## Setup

1. The user needs an Angelwood account: [start a 14-day free trial](https://app.angelwood.io/sign-up).
2. In Angelwood, open **Settings** and create an API key. Read permission is enough for lookups; creating clients or
   tasks needs **write** permission.
3. Store it as `ANGELWOOD_API_KEY`. Never ask the user to paste the key into chat, and never print it.

- Base URL: `https://angelwood-api-uzil.onrender.com/api/v1`
- Header on every request: `Authorization: Bearer $ANGELWOOD_API_KEY`
- OpenAPI description: `https://angelwood-api-uzil.onrender.com/api/v1/openapi.json`

Check the connection first:

```bash
curl -s https://angelwood-api-uzil.onrender.com/api/v1/me -H "Authorization: Bearer $ANGELWOOD_API_KEY"
```

It returns the organization, the user, their role and the key's `permissions`. A key acts inside that one
organization only.

## Endpoints

| Request | What it does | Parameters or body |
|---|---|---|
| `GET /me` | Tests the key | |
| `GET /clients` | Active clients, newest first | `search`, `created_after` |
| `GET /clients/{id}` | One client | |
| `POST /clients` | Creates a client (write) | `name` (required), `email`, `phone`, `address` |
| `GET /projects` | Projects, newest first | `client_id`, `status`, `created_after` |
| `GET /tasks` | Tasks, newest first | `project_id`, `status`, `created_after`, `completed_after` |
| `POST /tasks` | Adds a task to a project (write) | `project_id`, `title` (required), `description`, `priority`, `due_date` |
| `GET /invoices` | Invoices, newest first | `client_id`, `project_id`, `status`, `created_after`, `paid_after` |

Values: project `status` is `active`, `completed`, `on_hold` or `cancelled`; task `status` is `todo`, `in_progress`,
`completed` or `blocked`; task `priority` is `low`, `medium`, `high` or `critical`; invoice `status` is `draft`,
`sent`, `paid`, `overdue`, `partially_paid`, `refunded`, `partially_refunded` or `cancelled`. Times are ISO 8601, for
example `2026-03-31T00:00:00Z`.

## Paging

Lists return `{ "data": [...], "total": n, "has_more": bool, "next_offset": n | null }`. Use `limit` (1 to 100,
default 25) and `offset`. Before stating a count, use `total`; before stating a sum, page with `next_offset` until
`has_more` is false.

## Rules

- Resolve names to ids first: `GET /clients?search=` for a client id, `GET /projects?client_id=` for a project id.
  Never guess an id.
- Before `POST`, tell the user exactly what will be created and where, and wait for a yes. There is no undo through
  the API.
- `POST /clients` returns 409 `CONFLICT` with `details.client_id` when an active client already has that email. Offer
  the existing client instead of retrying.
- Invoice amounts come with a `currency` code. `balance_due` is `amount` minus completed payments.
- There is no "overdue only" filter: `status=overdue` returns invoices already marked overdue. For everything unpaid,
  also query `sent` and `partially_paid` and compare `due_date` with today.
- Tasks have no client filter. For one client's tasks, list that client's projects and query each `project_id`.
- On 401, the key is missing, revoked or expired, or its owner left the organization: ask the user to create a new
  key. On 403 `INSUFFICIENT_SCOPE`, the key is read-only. On 429, wait a minute (120 requests per minute per
  organization).
- Relay Angelwood's error `message` to the user as written.

## Examples

```bash
# Overdue invoices
curl -s "https://angelwood-api-uzil.onrender.com/api/v1/invoices?status=overdue&limit=100" \
  -H "Authorization: Bearer $ANGELWOOD_API_KEY"

# Add a task to project 42
curl -s -X POST https://angelwood-api-uzil.onrender.com/api/v1/tasks \
  -H "Authorization: Bearer $ANGELWOOD_API_KEY" -H "Content-Type: application/json" \
  -d '{"project_id": 42, "title": "Send the Q4 report", "priority": "high"}'
```

## Limits

- No updating or deleting: tasks cannot be completed, reassigned or edited, and invoices cannot be created or sent.
- Time entries, estimates, proposals, client requests and client updates are not available.

## Links

- [Angelwood](https://angelwood.io) · [Start a free trial](https://app.angelwood.io/sign-up) ·
  [Pricing](https://angelwood.io/pricing)
- Support: support@angelwood.io
