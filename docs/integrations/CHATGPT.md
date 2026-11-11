# ChatGPT

Connect ChatGPT to Conducktor so it can work with your workspace: your connectors
(including the Builds in your Stacks) and your Drive.

## What ChatGPT sees

ChatGPT connects to one MCP server, `https://mcp.conducktor.com/mcp`, and by default sees
two tools:

| Tool | What it does |
|------|--------------|
| `ask_conducktor` | Takes a plain-language request ("list my open Linear issues", "draft a reply to the last email from Sam"). Conducktor works out which connectors and tools to use, runs the steps, and asks a follow-up question when it needs one. |
| `send_to_conducktor` | Sends a file. Conducktor scans it, saves it to your Drive and returns a receipt. |

ChatGPT does not see your individual connector tools. Conducktor chooses them and runs
them through its governance checks (permissions, rate limits, argument validation), with
each call audit-logged.

## Before you start

- A Conducktor account. Each account owns one workspace; ChatGPT works with that workspace.
- At least one connector in your workspace (open **Integrations** in Conducktor), or a
  Stack installed from **Stacks**, so there is something to work with.
- A ChatGPT plan that lets you add remote MCP servers in Developer mode.

## Connect

These steps follow ChatGPT's settings as of October 2026; ChatGPT may rename them.

1. In ChatGPT, open **Settings → Security and login** and turn on **Developer mode**. In a
   ChatGPT workspace, it is under **Settings → Apps → Advanced settings**.
2. Open the **Plugins** page (**Apps & Connectors** in a ChatGPT workspace) and click **+**.
3. Name it `Conducktor`, paste `https://mcp.conducktor.com/mcp`, choose **OAuth**, and save.
4. When ChatGPT asks, sign in with your Conducktor account and approve the connection.

ChatGPT registers itself with Conducktor automatically (OAuth Dynamic Client Registration),
so you do not need a client ID or secret.

## Try it

Ask ChatGPT something that needs one of your connectors, for example:

```
Use Conducktor to show my open issues in Linear.
```

ChatGPT calls `ask_conducktor` with your request, and Conducktor answers with the result
or with a question.

## Manage the connection

- **See or revoke access.** In Conducktor, open your account page and go to the
  **Connected Apps** tab. Revoking a client ends its access; ChatGPT then has to sign in
  again.
- **Tokens.** Access tokens last 24 hours by default; the client renews them with its refresh token.
  See the [OAuth guide](../authentication/OAUTH_GUIDE.md#refresh-revoke-introspect).

## Troubleshooting

| Problem | What to check |
|---------|---------------|
| ChatGPT cannot connect | The URL is exactly `https://mcp.conducktor.com/mcp`, and you chose OAuth. Remove the server in ChatGPT and add it again. |
| ChatGPT keeps asking you to sign in | The client was revoked, or its refresh failed. Remove the server in ChatGPT and add it again. |
| A request says a connector is missing or not connected | Open **Integrations** in Conducktor and connect or reconnect that service. |

## Other MCP clients

Claude, Claude Code, Cursor, VS Code and any client that supports remote MCP servers with
OAuth connect the same way. Step-by-step instructions for each:
<https://www.conducktor.com/docs>.
