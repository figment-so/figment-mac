# Figment Mac for AI agents

Rent a Mac in the cloud from [Figment](https://figment.so) through an MCP server, an HTTP API or an
agent skill. Use it when a task needs macOS and no Mac is available: an Xcode build, the iOS Simulator,
Safari, a Mac app or AppleScript.

- No sign-in. The agent makes an account with one call.
- Prepaid. Your user pays one Stripe link for credit, and the agent spends that credit.
- By the hour or by the day. Stop the Mac when the task is done and billing stops.

Plans, prices and live availability: https://figment.so/api/agent/v1/offer
Full guide for agents: https://figment.so/agents.md
OpenAPI description: https://figment.so/api/agent/v1/openapi.json

## MCP server

Remote server, Streamable HTTP: `https://figment.so/api/mcp`

Tools: `figment_get_offer`, `figment_create_account`, `figment_get_account`, `figment_buy_credit`,
`figment_start_mac`, `figment_get_mac`, `figment_get_mac_access`, `figment_stop_mac`.

The first two tools need no key. The others take the key from `figment_create_account`, as an
`Authorization: Bearer <api_key>` header on the connection or as the `api_key` argument of the tool.

Claude Code:

```bash
claude mcp add --transport http figment https://figment.so/api/mcp
```

Other clients: add a remote (HTTP) MCP server with the URL above. This repository has the config files
that the common clients read: `.mcp.json`, `mcp.json` and `server.json`.

## Agent skill

`skills/figment-mac/SKILL.md` tells an agent how to rent a Mac step by step.

```bash
npx skills add figment-so/figment-mac
```

The same skill is served at `https://figment.so/.well-known/agent-skills/index.json`.

## Claude Code plugin

```bash
claude plugin marketplace add figment-so/figment-mac
claude plugin install figment-mac@figment
```

## What the agent does

1. Reads the offer and chooses a plan.
2. Makes an account and keeps the API key secret.
3. Asks its user to pay a Stripe link for credit (at least the first-purchase minimum in the offer).
4. Orders a Mac and waits until its state is `ready`. Setup is not instant.
5. Reads the SSH host and port, a one-time browser console link, and the Mac login when it is available.
6. Stops the Mac when the task is done.

The user can sign in on figment.so later, with the claim link the agent gives them, to see the Macs and
the credit in a browser. This is optional.

## Support

support@figment.so

## Licence

MIT. The Figment service has its own terms: https://figment.so/terms
