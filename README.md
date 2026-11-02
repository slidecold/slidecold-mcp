# slidecold-mcp

Official [MCP](https://modelcontextprotocol.io) server for [SlideCold](https://slidecold.com) (Instagram cold-DM outreach). It gives Claude, Cursor and any other MCP client all 49 SlideCold API operations as tools, over stdio. Built on the [`slidecold`](https://github.com/slidecold/slidecold-node) Node.js SDK. Node.js 18 or newer.

Create an API key (`cs_...`) at <https://slidecold.com/settings>. Give the server only the scopes you want the assistant to have; a read-only key (`*:read`) is a good start.

## Setup

**Claude Code**

```sh
claude mcp add slidecold -e SLIDECOLD_API_KEY=cs_... -- npx -y slidecold-mcp
```

**Claude Desktop** (Settings > Developer > Edit Config, `claude_desktop_config.json`)

```json
{
  "mcpServers": {
    "slidecold": {
      "command": "npx",
      "args": ["-y", "slidecold-mcp"],
      "env": { "SLIDECOLD_API_KEY": "cs_..." }
    }
  }
}
```

**Cursor** (`~/.cursor/mcp.json`, or `.cursor/mcp.json` in a project): the same `mcpServers` JSON as Claude Desktop.

**Other clients** (Windsurf, VS Code, Cline, Zed, ...): register a stdio server with command `npx`, arguments `-y slidecold-mcp` and the environment variable `SLIDECOLD_API_KEY`. With a global install (`npm install -g slidecold-mcp`) the command is just `slidecold-mcp`.

## Configuration

| Setting | Source |
| --- | --- |
| API key | `SLIDECOLD_API_KEY`, then `apiKey` in the config file |
| API URL | `SLIDECOLD_BASE_URL`, then `baseUrl` in the config file, then `https://slidecold.com/api/v1` |

The config file is written by `slidecold login` from the [SlideCold CLI](https://github.com/slidecold/slidecold-cli): `$XDG_CONFIG_HOME/slidecold/config.json` (default `~/.config/slidecold/config.json`), or `%APPDATA%\slidecold\config.json` on Windows. If you have logged in with the CLI you can leave out `SLIDECOLD_API_KEY`.

```json
{ "apiKey": "cs_...", "baseUrl": "https://slidecold.com/api/v1" }
```

## Tools

One tool per API operation, named after its `operationId` in snake case. Each description gives the summary, the required API key scope and the HTTP endpoint.

| Group | Tools |
| --- | --- |
| Campaigns (8) | `list_campaigns`, `create_campaign`, `get_campaign`, `update_campaign`, `delete_campaign`, `get_campaign_stats`, `list_campaign_recipients`, `retry_campaign` |
| Audiences (10) | `list_audiences`, `create_audience`, `get_audience`, `update_audience`, `delete_audience`, `list_audience_members`, `import_audience_members`, `export_audience`, `start_audience_scrape`, `stop_audience_scrape` |
| Conversations (8) | `list_conversations`, `refresh_conversations`, `get_conversation`, `update_conversation`, `send_conversation_reply`, `send_conversation_photo`, `export_conversation_lead`, `delete_conversation_message` |
| Accounts (14) | `list_accounts`, `create_account`, `get_login_status`, `get_accounts_proxy_health`, `verify_account_2fa`, `verify_account_challenge`, `get_account`, `update_account`, `delete_account`, `check_account_session`, `relogin_account`, `set_account_listening`, `get_account_warmup_history`, `get_account_warmup_status` |
| Proxies (5) | `list_proxies`, `create_proxy`, `update_proxy`, `delete_proxy`, `test_proxy` |
| Folders (4) | `list_folders`, `create_folder`, `update_folder`, `delete_folder` |

Arguments are the path parameters (e.g. `id`), the query parameters, and `body` for the JSON request body. Upload tools (`send_conversation_photo`, `import_audience_members`) take a local `file_path`. API errors come back as tool errors carrying the API's message, e.g. `SlideCold API error 404: Campaign not found`.

### Annotations

Every tool carries MCP annotations so clients can ask before risky calls:

- `readOnlyHint: true` on every GET tool (`list_*`, `get_*`, `export_audience`).
- `destructiveHint: true` on every DELETE tool (`delete_*`, `stop_audience_scrape`) and on the two tools that send a DM to a real person and cannot be undone: `send_conversation_reply` and `send_conversation_photo`. Other write tools have `destructiveHint: false`.
- `idempotentHint: true` on GET, PUT, PATCH and DELETE tools.

## Scopes

Each API key carries scopes `<resource>:<read|write>` for `accounts`, `audiences`, `campaigns` and `conversations`; `:write` also grants `:read`. Proxy tools use the `accounts` scopes; folder tools use the scope of the folder's type. Read tools need `:read`, everything else needs `:write`. A tool called without the scope returns a 403 tool error such as `This API key is missing the campaigns:write scope`.

## MCP Registry

[`server.json`](server.json) describes this server for the [official MCP Registry](https://registry.modelcontextprotocol.io) as `io.github.slidecold/slidecold-mcp`; the matching `mcpName` in `package.json` proves ownership of the npm package. After a version is on npm, publish it to the registry with [`mcp-publisher`](https://github.com/modelcontextprotocol/registry):

```sh
mcp-publisher login github   # as a member of the slidecold GitHub org
mcp-publisher publish
```

Bump `version` in `server.json` (top level and in `packages`) together with `package.json`; the release workflow checks they match the tag.

## Development

```sh
npm install
npm run build
npm run typecheck
npm test
```

The tests start the built stdio server (`dist/bin.js`), list its 49 tools and call them against a local mock API.

To develop against an unreleased SDK, build it in a sibling checkout of [`slidecold-node`](https://github.com/slidecold/slidecold-node) and link it without touching `package.json`:

```sh
(cd ../slidecold-node && npm install && npm run build)
npm install --no-save ../slidecold-node   # node_modules/slidecold -> ../slidecold-node
```

Releases are cut by bumping the versions in `package.json` and `server.json`, adding a CHANGELOG entry and pushing a matching `vX.Y.Z` tag; CI tests, publishes to npm with provenance and creates the GitHub release.

## License

MIT
