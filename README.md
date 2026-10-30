<p align="center"><img src="assets/logo.svg" width="72" height="72" alt="Amplify logo"></p>

# Amplify MCP server

Make browser games from your AI agent. Describe a game, and Amplify builds a playable browser version in your Amplify account. Keep changing it by prompt, follow the build, preview it, and ask to publish it at its own address. Amplify builds web apps and websites the same way.

This repository has no server code. It holds the connection config for Amplify's hosted MCP server, plus ready-made plugin and extension manifests for Claude Code, Cursor and Gemini CLI.

| | |
|---|---|
| Server URL | `https://app.amplify.is/mcp` |
| Transport | Streamable HTTP (stateless) |
| Auth | OAuth 2.1 with PKCE (S256) and dynamic client registration. No API key. |
| Registry name | `is.amplify/amplify` ([official MCP Registry](https://registry.modelcontextprotocol.io/v0/servers?search=is.amplify/amplify)) |
| Docs | https://amplify.is/data |

You need an Amplify account. You can create one at [amplify.is](https://amplify.is).

## Connect

Your client registers itself and opens Amplify's sign-in and consent page in the browser the first time it connects. There is nothing to paste except the URL.

### Claude Code

```bash
claude mcp add --transport http amplify https://app.amplify.is/mcp
```

Or install this repository as a plugin:

```text
/plugin marketplace add Gudmundur76/amplify-mcp
/plugin install amplify@amplify
```

Then run `/mcp` and choose **amplify** to sign in.

### Claude (web and desktop)

Settings → Connectors → Add custom connector. Name: `Amplify`, URL: `https://app.amplify.is/mcp`.

### Cursor

Add to `~/.cursor/mcp.json` (or `.cursor/mcp.json` in a project):

```json
{
  "mcpServers": {
    "amplify": {
      "url": "https://app.amplify.is/mcp"
    }
  }
}
```

This repository is also a Cursor plugin (`.cursor-plugin/plugin.json` and `mcp.json`).

### VS Code (GitHub Copilot)

Add to `.vscode/mcp.json`:

```json
{
  "servers": {
    "amplify": {
      "type": "http",
      "url": "https://app.amplify.is/mcp"
    }
  }
}
```

### Gemini CLI

```bash
gemini extensions install https://github.com/Gudmundur76/amplify-mcp
```

Or add it to `~/.gemini/settings.json`:

```json
{
  "mcpServers": {
    "amplify": {
      "httpUrl": "https://app.amplify.is/mcp"
    }
  }
}
```

Run `/mcp auth amplify` in Gemini CLI to sign in.

### Windsurf

```json
{
  "mcpServers": {
    "amplify": {
      "serverUrl": "https://app.amplify.is/mcp"
    }
  }
}
```

### Other clients

Any MCP client that supports remote Streamable HTTP servers with OAuth can connect with just `https://app.amplify.is/mcp`. For a stdio-only client, use a bridge such as `npx -y mcp-remote https://app.amplify.is/mcp`.

## Sign-in and consent

When you connect, Amplify asks you to sign in and shows a consent page where you choose:

- **Permissions** (scopes): `read`, `build` and `publish`. Grant only what you need.
- **Projects**: all projects, only the projects this connection creates, or a single project.
- **Daily credit cap** for this connection.

You can disconnect a client at any time under **Account → Connected apps** in Amplify. Every tool call is logged. Access tokens last one hour, and your client refreshes them for up to 30 days.

## Scopes

| Scope | What it allows |
|---|---|
| `read` | Read your account, projects, files, versions, builds, credits and approvals. |
| `build` | Create and edit projects. This spends your credits, always with a two-step confirmation. |
| `publish` | Ask to publish a project and list custom domains. Publishing waits for your approval in the app. Needs a verified email. |

## Credits and the two-step confirmation

Building and editing spend credits from your own Amplify account. Every spending call is two-step:

1. The agent calls `create_project`, `create_build` or `edit_project`. Nothing runs yet. Amplify answers `needs_confirmation` with the price and a single-use `confirmToken`.
2. The agent shows you the price. Only after you agree does it call `confirm_action` with the token. A token works once, on that connection only, within 10 minutes.

Spending also counts against the connection's daily credit cap. When the cap or your balance runs out, the call stops and says how much is needed and where to top up.

## Tools

The server shows only the tools your granted scopes allow.

**Read** (`read`, read-only)

| Tool | What it does |
|---|---|
| `get_account` | Who this connection acts as: user, active org, plan, credit balance, and the scopes granted. |
| `list_projects` | The org's projects, newest first. A connection limited to some projects sees only those. |
| `get_project` | One project: status, kind, files, model, published address, preview path. |
| `read_project_file` | Read one text file of a project, such as `index.html` or `app.js`. |
| `list_versions` | Version history of a project. |
| `get_publish_check` | Run the publish checks for a project without publishing, to see what would block it. |
| `list_published_sites` | Published projects with their public slug and address. |
| `list_approvals` | The approval queue: publish requests and other actions waiting for a person. Approving happens in the app. |
| `get_credits` | Credit balance and recent ledger entries. |
| `get_connection` | This connection's project access, daily credit cap, today's spend and what is left. |
| `get_usage_today` | Today's model usage against the daily cap. |
| `get_build_status` | Where a build is (queued, building, ready or failed), with the preview or public address when ready. |
| `get_result` | The delivery of a finished build: a single-use claim link and a preview address. |

**Build** (`build`, spends credits, two-step)

| Tool | What it does |
|---|---|
| `create_project` | Create a game, web app or website from a prompt and build it. |
| `create_build` | Same, for apps that order builds for their own customers. Answers as soon as the build starts. An `idempotencyKey` makes a retry return the same build. |
| `edit_project` | Apply a follow-up prompt to a project, creating a new version. An edit that changes nothing is free. |
| `confirm_action` | Run a spending action you confirmed, using its `confirmToken`. |

**Publish** (`publish`)

| Tool | What it does |
|---|---|
| `publish_project` | Ask to publish a project at a public address. It queues an approval that you confirm in the app. Nothing goes live until you do. |
| `list_custom_domains` | Custom domains on a project and their status (read-only). |

The server has no delete, billing or payment tools.

## Example prompts

- "Make a one-button endless runner where a fox jumps over logs, with a high score."
- "Change the fox to a penguin and make the logs icebergs."
- "What's the status of my last build? Send me the preview link."
- "Run the publish check on my runner game and tell me what would block it."

## Files in this repository

| File | For |
|---|---|
| `server.json` | Official MCP Registry entry (`is.amplify/amplify`) |
| `.mcp.json` | Claude Code and other clients that read a project `.mcp.json`; also read by cursor.directory |
| `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json` | Claude Code plugin and a one-plugin marketplace |
| `.cursor-plugin/plugin.json`, `mcp.json` | Cursor plugin |
| `gemini-extension.json`, `GEMINI.md` | Gemini CLI extension |
| `assets/` | Amplify logo (SVG and 180×180 PNG) |

## Links

- Website: https://amplify.is
- Docs and data handling: https://amplify.is/data
- Privacy: https://amplify.is/privacy
- Terms: https://amplify.is/terms
- Contact: hello@amplify.is

## Licence

The files in this repository are under the [MIT licence](LICENSE). The Amplify service is covered by its [terms](https://amplify.is/terms).
