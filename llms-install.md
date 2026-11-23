# Install Figment Mac (for an AI agent that sets this up)

Figment Mac is a remote MCP server. There is nothing to download or build.

1. Add a remote MCP server with transport "Streamable HTTP" and this URL: `https://figment.so/api/mcp`
   - Claude Code: `claude mcp add --transport http figment https://figment.so/api/mcp`
   - A client with a JSON config: `{"mcpServers":{"figment":{"type":"http","url":"https://figment.so/api/mcp"}}}`
     (some clients name the type `streamable-http`; `mcp.json` in this repository uses that name).
2. No API key is needed to connect. Call the tool `figment_get_offer` to check the connection. It returns
   three Mac plans with prices.
3. The tool `figment_create_account` returns an `api_key`. Store it as a secret named `FIGMENT_API_KEY`.
   Then either add the header `Authorization: Bearer <api_key>` to the MCP server config, or pass
   `api_key` to each tool. Never write the key into a file that is committed.
4. Do not order a Mac during setup. Ordering costs money and needs the user's agreement.

Guide: https://figment.so/agents.md
