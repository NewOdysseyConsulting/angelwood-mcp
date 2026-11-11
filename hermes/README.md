# Angelwood for Hermes Agent

[Angelwood](https://angelwood.io) is a client portal and client operations platform for agencies, consultancies and
other service businesses. It gives each client a branded portal for updates, requests and files, and runs the back
office behind it: projects and tasks, time tracking, estimates, proposals and invoices.

This directory is a portable Agent Plugins v1 package for [Hermes Agent](https://hermes-agent.nousresearch.com)
(`plugin.json`, `mcp.json` and `skills/`). It is configuration only: no local code runs. It gives Hermes:

- **The Angelwood MCP server** (`angelwood`, Streamable HTTP at `https://mcp.angelwood.io/mcp`), hosted
  by Angelwood.
- **The `angelwood:angelwood` skill**, which tells the agent when and how to use the tools.

## Install

```bash
hermes plugins install angelwood
hermes plugins enable angelwood
```

## Connect your Angelwood account

Sign-in is OAuth with your own Angelwood account. Add this to your profile's `config.yaml`:

```yaml
mcp_servers:
  angelwood:
    url: https://mcp.angelwood.io/mcp
    auth: oauth
    trust: untrusted
```

Then run `hermes mcp login angelwood` in a terminal. Hermes opens the Angelwood sign-in; sign in and approve. Hermes
stores and refreshes the token, and the plugin never sees it. This entry replaces the package's own `angelwood`
server, so keep `trust: untrusted` in it.

You need an Angelwood account: [start a 14-day free trial](https://app.angelwood.io/sign-up). API keys are not
accepted by the MCP server.

## What you can ask

- "Which invoices are overdue, and how much is outstanding in total?"
- "What's still open for Northwind this week?"
- "Give me an overview of Acme: contact details, active projects and what they owe."
- "Which active projects have the most open tasks?"
- "What's assigned to me and due by Friday?"

## Tools

| Tool | Changes anything? |
|---|---|
| `angelwood_whoami`, `angelwood_search_clients`, `angelwood_get_client_overview` | No (read-only) |
| `angelwood_list_projects`, `angelwood_list_tasks`, `angelwood_list_invoices` | No (read-only) |

Every tool is read-only: the assistant can look things up in Angelwood, and cannot create, change or send anything.
The package still marks the server `untrusted`, so if Angelwood adds a tool that changes something, Hermes will ask
you before running it and refuse it in cron jobs and other runs where nobody can answer.

## What to know before installing

- **Network calls.** Every tool call goes to Angelwood's MCP server at `mcp.angelwood.io` (operated by New
  Odyssey). The OAuth sign-in also talks to Angelwood's sign-in service at `clerk.angelwood.io`. Hermes contacts
  nothing else for this plugin. Angelwood's privacy policy: https://angelwood.io/privacy.
- **What it reads.** Tool results contain your organization's client names and contact details, project and task
  titles and descriptions, and invoice amounts and balances. They are returned to Hermes and to the model provider
  you use with it. Each call runs inside one organization you are an active member of.
- **Credentials.** The plugin stores none. Hermes's MCP client keeps and refreshes the OAuth token in your Hermes
  profile. The plugin reads no files outside its own directory.
- **No local code, no background processes, no telemetry** from the plugin. The tool set is served by Angelwood's
  hosted server at run time, so it is not fixed by the pinned commit.

## Links

- Website: https://angelwood.io
- Start a free trial: https://app.angelwood.io/sign-up
- Pricing: https://angelwood.io/pricing
- Privacy: https://angelwood.io/privacy
- Support: support@angelwood.io
- Licence: MIT (see [LICENSE](../LICENSE))
