# Figment Mac for AI agents

[![mcptry](https://mcptry.com/badge/so.figment/mac)](https://mcptry.com/servers/so.figment/mac)

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
`figment_start_mac` (a new Mac), `figment_get_mac`, `figment_get_mac_access`, `figment_stop_mac`,
`figment_resume_mac` (start a stopped Mac again), `figment_delete_mac`, `figment_send_feedback`.

The first two tools and `figment_send_feedback` need no key. The others take the key from `figment_create_account`, as an
`Authorization: Bearer <api_key>` header on the connection or as the `api_key` argument of the tool.

Claude Code:

```bash
claude mcp add --transport http figment https://figment.so/api/mcp
```

Other clients: add a remote (HTTP) MCP server with the URL above. This repository has the config files
that the common clients read: `.mcp.json`, `mcp.json` and `server.json`.

## Agent skills

- `skills/figment-mac/SKILL.md` tells an agent how to rent a Mac step by step.
- `skills/needs-macos/SKILL.md` starts when an agent on Linux or Windows hits a task that needs macOS,
  for example "xcodebuild: command not found" or a request to build for iOS. It says what a new Mac can
  run at once (Apple's Command Line Tools) and what needs Xcode, then how to copy the project to the
  Mac over SSH, build, and bring the results back.

```bash
npx skills add figment-so/figment-mac
```

The same skill is served at `https://figment.so/.well-known/agent-skills/index.json`.

## Claude Code plugin

```bash
claude plugin marketplace add figment-so/figment-mac
claude plugin install figment-mac@figment
```

## Codex plugin

```bash
codex plugin marketplace add figment-so/figment-mac
codex plugin add figment-mac@figment
```

Start a new Codex session after the install. The plugin has the MCP server and both skills.

## Other agent tools

| Tool | How to add Figment Mac |
| --- | --- |
| Gemini CLI | `gemini extensions install https://github.com/figment-so/figment-mac` (reads `gemini-extension.json`) |
| VS Code, Copilot CLI | `code --add-mcp '{"name":"figment","type":"http","url":"https://figment.so/api/mcp"}'`, or add this repository as an agent plugin marketplace |
| GitHub Copilot coding agent | Repository settings, Copilot, MCP servers: `{"mcpServers":{"figment":{"type":"http","url":"https://figment.so/api/mcp","headers":{"Authorization":"Bearer $COPILOT_MCP_FIGMENT_API_KEY"},"tools":["*"]}}}` with the key in the secret `COPILOT_MCP_FIGMENT_API_KEY` |
| Augment (Auggie) | `auggie plugin marketplace add figment-so/figment-mac` |
| Windsurf | `mcp_config.json`: `{"mcpServers":{"figment":{"serverUrl":"https://figment.so/api/mcp"}}}` |
| Zed | Settings, AI, MCP Servers, Add Remote Server: `https://figment.so/api/mcp` |
| OpenCode | `opencode.json`: `{"mcp":{"figment":{"type":"remote","url":"https://figment.so/api/mcp","oauth":false}}}` |
| Factory (droid) | `droid mcp add figment https://figment.so/api/mcp --type http` |
| Kiro | `.kiro/settings/mcp.json`: `{"mcpServers":{"figment":{"url":"https://figment.so/api/mcp"}}}` |
| Warp, Cursor and other clients | They read `.mcp.json` or `mcp.json` from this repository, or take the URL above as a remote MCP server |
| Perplexity, Mistral Le Chat, Grok | Add a custom connector with the URL `https://figment.so/api/mcp` and no authentication. Tools that need an account take `api_key` |

## GitHub Actions

To run one CI job step on a rented Mac, use the action
[figment-so/run-on-mac](https://github.com/figment-so/run-on-mac). It orders the Mac, runs the step
over SSH, copies the files back and ends the Mac, also when the job fails or is cancelled.

## What the agent does

1. Reads the offer and chooses a plan.
2. Makes an account and keeps the API key secret.
3. Asks its user to pay a Stripe link for credit (at least the first-purchase minimum in the offer).
4. Orders a Mac and waits until its state is `ready`. Setup is not instant.
5. Reads the SSH host and port, a one-time browser console link, and the Mac login when it is available.
6. Stops the Mac when the task is done.

The user can sign in on figment.so later, with the claim link the agent gives them, to see the Macs and
the credit in a browser. This is optional.

## Feedback

An agent can tell Figment what happened with `figment_send_feedback`, or with
`POST https://figment.so/api/agent/v1/feedback`. No key is needed. The founder reads each message.

## Support

support@figment.so

## Licence

MIT. The Figment service has its own terms: https://figment.so/terms
