# Angelwood MCP server

[Angelwood](https://angelwood.io) is a client portal and client operations platform for agencies, consultancies and
other service businesses. It gives each client a branded portal for updates, requests and files, and runs the back
office behind it: projects and tasks, time tracking, estimates, proposals and invoices.

This repo holds the configuration, skill and rules for using Angelwood from AI assistants and editors (Claude, ChatGPT,
Cursor, Hermes Agent and any MCP client). The server itself is hosted by Angelwood.

**Server (Streamable HTTP):** `https://mcp.angelwood.io/mcp`

## Connect

- **Claude / ChatGPT:** add `https://mcp.angelwood.io/mcp` as a custom connector.
- **Hermes Agent:** the [`hermes/`](hermes/) package; see [`hermes/README.md`](hermes/README.md) to install and
  sign in.
- **Cursor:** add [`.cursor/mcp.json`](.cursor/mcp.json) from this repo (or copy the entry below) to your project or
  global MCP settings.
- **Any MCP client:** use the entry in [`mcp.json`](mcp.json).

```json
{
  "mcpServers": {
    "angelwood": { "type": "http", "url": "https://mcp.angelwood.io/mcp" }
  }
}
```

On first use your client opens the Angelwood sign-in. Sign in and approve, and the assistant can then read the same
data you can see in Angelwood. You need an Angelwood account:
[start a 14-day free trial](https://app.angelwood.io/sign-up).

## What you can ask

- "Which invoices are overdue, and how much is outstanding in total?"
- "What's still open for Northwind this week?"
- "Give me an overview of Acme: contact details, active projects and what they owe."
- "Which active projects have the most open tasks?"
- "List every task that is blocked, soonest due date first."

## Tools

All tools are read-only. An assistant can look things up in Angelwood; it cannot create, change or send anything.

| Tool | What it does |
| --- | --- |
| `angelwood_whoami` | The signed-in user and the organizations they belong to, with their role in each. |
| `angelwood_search_clients` | Active clients, optionally filtered by name or email. |
| `angelwood_get_client_overview` | One client's contact details, recent projects and invoice summary, including the balance still owed. |
| `angelwood_list_projects` | Projects with client, status, dates, hour budget and open task count. |
| `angelwood_list_tasks` | Tasks across projects, soonest due first. Open tasks by default. |
| `angelwood_list_invoices` | Invoices with amount, amount paid and balance due. Filter by status, client or project, or overdue only. |

Every call runs inside one organization you are an active member of. If you belong to several, the assistant asks
which one, or you can name it. Details are in [`skills/angelwood/SKILL.md`](skills/angelwood/SKILL.md).

## Install the skill

```bash
npx skills add NewOdysseyConsulting/angelwood-mcp
```

## Links

- **Website:** [angelwood.io](https://angelwood.io)
- **Start a free trial:** [app.angelwood.io/sign-up](https://app.angelwood.io/sign-up)
- **Pricing:** [angelwood.io/pricing](https://angelwood.io/pricing)
- **Product:** [client portal for agencies](https://angelwood.io/client-portal-for-agencies) ·
  [client onboarding software](https://angelwood.io/client-onboarding-software) ·
  [features](https://angelwood.io/features) · [integrations](https://angelwood.io/integrations)
- **Who it's for:** [angelwood.io/for](https://angelwood.io/for)
- **Free resources:** [templates](https://angelwood.io/templates) · [tools](https://angelwood.io/tools) ·
  [blog](https://angelwood.io/blog) · [glossary](https://angelwood.io/glossary)
- **Comparisons:** [angelwood.io/compare](https://angelwood.io/compare)
- **Privacy:** [angelwood.io/privacy](https://angelwood.io/privacy) · **Terms:** [angelwood.io/terms](https://angelwood.io/terms)
- **Support:** support@angelwood.io

Angelwood is made by [New Odyssey](https://new-odyssey.com).
