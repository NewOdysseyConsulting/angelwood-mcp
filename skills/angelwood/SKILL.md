---
name: angelwood
description: Look up an agency's clients, projects, tasks and invoices in Angelwood (angelwood.io) through its read-only MCP server. Use when the user asks which invoices are overdue or unpaid, what a client owes, what tasks are open or due for a client or project, what's assigned to them, which projects are active, or for a summary of one client's account in Angelwood.
---

# Angelwood

Angelwood is a client portal and client-operations platform for agencies, consultancies and other service businesses:
branded client portals, client updates and requests, projects and tasks, plus a back office for time tracking,
invoices, estimates and proposals. This skill reads an organization's clients, projects, tasks and invoices through
Angelwood's remote MCP server. It is read-only: it answers questions, it doesn't change anything.

## When to use

- "Which invoices are overdue?", "who owes us money?", "what does Acme still owe?"
- "What's open for Acme this week?", "what's due by Friday?", "what's assigned to me?"
- "Which projects are active?", "how many open tasks does the website project have?"
- "Give me a summary of the Acme account", "find the client with this email"

Don't use it to create or edit anything in Angelwood (see Limits), for organizations the user isn't a member of, or
for data the tools don't return: time entries, estimates, proposals, client requests, client updates and portal
content are not available here.

## Setup

MCP server (Streamable HTTP): `https://mcp.angelwood.io/mcp`

Sign-in is OAuth with the user's own Angelwood account. Add the server URL with no headers; on first use the client
opens a browser sign-in, and the agent then sees the organizations that user is an active member of. API keys are not
accepted.

Example MCP server entry:

```json
{
  "mcpServers": {
    "angelwood": {
      "type": "http",
      "url": "https://mcp.angelwood.io/mcp"
    }
  }
}
```

The user needs an Angelwood account first. If a tool says no active Angelwood account is linked to the sign-in, the
user should sign in to the Angelwood web app once and then reconnect. No account yet:
[start a 14-day free trial](https://app.angelwood.io/sign-up).

## Tools

All six tools are read-only. Every tool except `angelwood_whoami` also accepts `organization_id` (see Organizations)
and returns the `organization` (id and name) it ran against.

| Tool | What it does |
|---|---|
| `angelwood_whoami` | The signed-in user and the organizations they belong to, with their role in each. |
| `angelwood_search_clients` | Active clients, optionally matched by name or email. Use it to find a `client_id`. |
| `angelwood_get_client_overview` | One client: contact details, recent projects and an invoice summary with the outstanding balance. |
| `angelwood_list_projects` | Projects with client, status, dates, hour budget and open task count. |
| `angelwood_list_tasks` | Tasks across projects, soonest due first. Open tasks only unless a status is given. |
| `angelwood_list_invoices` | Invoices with amount, amount paid, balance due and days overdue. |

### `angelwood_whoami`

No inputs. Returns `user` (`id`, `name`, `email`) and `organizations` (each `id`, `name`, `role`).

### `angelwood_search_clients`

- `query` (optional, 1 to 100 characters): case-insensitive text matched against client name and email. Omit to list
  every active client.
- `limit`, `offset`: see Pagination.

Returns `clients` sorted by name, each with `id`, `name`, `email`, `phone` and `project_count`. Only active clients
are listed.

### `angelwood_get_client_overview`

- `client_id` (required): from `angelwood_search_clients`.

Returns:

- `client`: `id`, `name`, `email`, `phone`, `address`, `is_active`, `default_hourly_rate`.
- `projects`: the 25 most recently updated, each with `id`, `name`, `status`, `start_date`, `end_date`.
  `project_count` is the client's full total; if it's more than 25, use `angelwood_list_projects` with `client_id`
  to see the rest.
- `invoices.by_status`: for each invoice status the client has, `count` and `total_amount`.
- `invoices.outstanding_balance`: what is still owed on sent, overdue and partially paid invoices, after completed
  payments.

### `angelwood_list_projects`

- `query` (optional, 1 to 100 characters): case-insensitive text matched against the project name.
- `client_id` (optional): only this client's projects.
- `status` (optional): `active`, `completed`, `on_hold` or `cancelled`. Omit for all statuses.
- `limit`, `offset`: see Pagination.

Returns `projects`, most recently updated first, each with `id`, `name`, `status`, `client` (`id`, `name`, or null),
`start_date`, `end_date`, `budget_hours` and `open_task_count` (tasks that are todo, in progress or blocked).

### `angelwood_list_tasks`

- `project_id` (optional): only tasks in this project, from `angelwood_list_projects`.
- `status` (optional): `todo`, `in_progress`, `completed` or `blocked`. Omit for all open tasks (todo, in progress
  and blocked); completed tasks are returned only when you pass `status: "completed"`.
- `due_on_or_before` (optional, `YYYY-MM-DD`): only tasks due on or before this date. This includes tasks already
  past due, and excludes tasks with no due date.
- `assigned_to_me` (optional, default false): only tasks assigned to the signed-in user.
- `limit`, `offset`: see Pagination.

Returns `tasks`, soonest due date first with undated tasks last, each with `id`, `title`, `description` (cut to 300
characters), `status`, `due_date`, `project` (`id`, `name`), `client_name`, `assigned_to_me`, `estimated_hours` and
`logged_hours`.

There is no client filter on tasks. For one client's tasks, list that client's projects and call
`angelwood_list_tasks` once per `project_id`. The result says whether a task is assigned to the signed-in user, not
who else it is assigned to.

### `angelwood_list_invoices`

- `status` (optional): `draft`, `sent`, `paid`, `overdue`, `partially_paid`, `refunded`, `partially_refunded` or
  `cancelled`. Ignored when `overdue_only` is true.
- `client_id` (optional): only this client's invoices.
- `project_id` (optional): only this project's invoices.
- `overdue_only` (optional, default false): only unpaid invoices (sent, overdue or partially paid) whose due date
  has passed.
- `limit`, `offset`: see Pagination.

Returns `invoices`, newest invoice date first, each with `id`, `invoice_number`, `status`, `client` (`id`, `name`),
`project` (`id`, `name`), `amount`, `amount_paid` (completed payments), `balance_due`, `invoice_date`, `due_date`,
`paid_date` and `days_overdue` (0 unless the invoice is unpaid and past due).

## Organizations

Every call runs inside exactly one organization the user is an active member of.

- **One organization:** leave `organization_id` out; it is used automatically.
- **Several:** a call without `organization_id` fails with a message listing the options. Call `angelwood_whoami`,
  ask the user which organization they mean if it isn't clear from the conversation, then pass that
  `organization_id` on every call. Don't guess, and don't merge results from different organizations unless the user
  asks for that.
- Ids belong to one organization. A `client_id` or `project_id` from one organization finds nothing in another.

## Pagination

The four list tools take `limit` (1 to 50, default 20) and `offset` (default 0), and return `total`, `has_more` and
`next_offset`.

- `total` is the number of matches across all pages. Use it to answer "how many" without fetching every page.
- While `has_more` is true, call again with `offset` set to `next_offset`. `next_offset` is null on the last page.
- Sums are not provided across pages. Before stating a total (for example, the full overdue amount), fetch every
  page, or say that the figure covers only the first N results.

## Rules

- **Resolve names to ids first.** Users name clients and projects; the tools filter by id. Use
  `angelwood_search_clients` or `angelwood_list_projects` with `query`, and if several match, ask which one.
- **Use `overdue_only` for "overdue".** It finds every unpaid invoice past its due date. `status: "overdue"` returns
  only invoices already marked with that status and can miss sent or partially paid invoices that are past due.
- **An empty list is not an error.** Filtering by a `client_id` or `project_id` that doesn't exist in the
  organization returns no results rather than an error. Check the id before telling the user there is nothing.
- **Report amounts as given.** Amounts are plain numbers with no currency field. Don't add a currency symbol unless
  the user has told you the currency.
- **Dates are `YYYY-MM-DD`.** Turn "this week" or "by Friday" into a concrete date for `due_on_or_before`, and tell
  the user which date you used.
- **Errors are Angelwood's own words.** A failed call returns a plain message saying what to do next (pass an
  `organization_id`, find the right client id, reconnect). Follow it or relay it; if it reports an internal error,
  try again once shortly, then tell the user.

## Example requests

- "Which invoices are overdue?" → `angelwood_list_invoices` with `overdue_only: true`; report client, invoice
  number, `balance_due` and `days_overdue`, paging while `has_more` is true.
- "What does Acme still owe us?" → `angelwood_search_clients` with `query: "Acme"`, then
  `angelwood_get_client_overview` and read `invoices.outstanding_balance`.
- "What's open for Acme this week?" → `angelwood_search_clients`, `angelwood_list_projects` with `client_id`, then
  `angelwood_list_tasks` for each `project_id` with `due_on_or_before` set to the last day of the week. Mention that
  tasks with no due date are left out, or run it again without the date to include them.
- "What's on my plate?" → `angelwood_list_tasks` with `assigned_to_me: true`.
- "Which projects are on hold?" → `angelwood_list_projects` with `status: "on_hold"`.
- "Show me Acme's unpaid invoices." → `angelwood_search_clients`, then `angelwood_list_invoices` with `client_id`,
  once each for `status: "sent"`, `"overdue"` and `"partially_paid"`.
- "Which organizations am I in?" → `angelwood_whoami`

## Limits

- **Read-only.** The tools cannot create, edit, send or delete anything: no new clients, projects or tasks, no
  status changes, no sending or marking invoices paid, no logging time. For any of that, tell the user it has to be
  done in the Angelwood app.
- **Four kinds of record.** Clients, projects, tasks and invoices only. Invoice line items, individual payments,
  time entries, estimates, proposals, client requests, client updates, files and other team members' details are
  not returned.
- **Inactive clients don't appear in search.** `angelwood_get_client_overview` still returns one if you already
  have its id.
- **Long task descriptions are cut** to 300 characters.

## Links

- [Angelwood](https://angelwood.io)
- [Start a free trial](https://app.angelwood.io/sign-up)
- [Pricing](https://angelwood.io/pricing) (plans from $99/month, 14-day free trial)
- Support: support@angelwood.io
