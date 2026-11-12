# FAQ

Short answers for teams connecting an AI client to Conducktor, or connecting a service to
it.

## What is Conducktor?

Your workspace, built around you: one place to build, connect and run your own tools,
which you can use in the workspace or through the AI assistant you prefer. What you build
lives in a Stack, which brings together the Builds, connections, tools and Assistants
needed for a purpose. Add Stacks from Conducktor and other creators from the Stack library.
Everything with endpoints is a connector, and outside AI clients reach the workspace
through `ask_conducktor`. See the [README](README.md).

## How do I connect an AI client?

Point it at `https://mcp.conducktor.com/mcp` and choose OAuth. The client registers itself
(Dynamic Client Registration) and opens Conducktor's sign-in page; sign in and approve.

- ChatGPT: [ChatGPT guide](docs/integrations/CHATGPT.md).
- Claude, Claude Code, Cursor, VS Code and other clients: <https://www.conducktor.com/docs>.
- Your own agent: use the authorization code flow with PKCE
  ([OAuth guide](docs/authentication/OAUTH_GUIDE.md#part-1-connecting-to-conducktor)).

To test, run the MCP Inspector and connect it to the same URL:

```bash
npx @modelcontextprotocol/inspector
```

## What tools does a client see?

Conducktor's gateway tools: `ask_conducktor` for requests in plain language,
`send_to_conducktor` for files, and `conducktor_vault` to ask for a user's permission to
sign in to a website with one of their saved logins. Conducktor picks the connector tools
for each request itself. See
[Platform APIs § MCP endpoint](docs/developers/PLATFORM_APIS.md#mcp-endpoint).

## How do I set up my workspace?

Register an account; it comes with its own workspace. Each account owns exactly one
workspace, and other users are not added to it. Then connect services in **Integrations**
and install Stacks from **Stacks**.

## How do I add a service that only has a REST API?

Use API-Bridge. Give it an OpenAPI 3.0, OpenAPI 3.1 or Swagger 2.0 spec (JSON or YAML, as a
file, inline or by URL), or a SOAP WSDL; it generates tools from the spec's operations, and you
attach it to your workspace as a connection. See
[Platform APIs § API-Bridge](docs/developers/PLATFORM_APIS.md#api-bridge-openapi-to-tools).

## How do I run my own code?

Upload it as a Build. Each Build lives in a Stack, and its HTTP operations become that
Stack's connector tools. Start with [Getting started](docs/developers/GETTING_STARTED.md).

## How are secrets handled?

Credentials are stored encrypted, and the UI shows masked values, so you can keep a stored
credential or replace it without seeing it. See [SECURITY.md](SECURITY.md).

## How do I publish a Stack for other people?

Upload it to your workspace as a [Stack package](docs/developers/STACK_PACKAGE.md) (a ZIP with
`stack.yaml` at its root), then send it for review from the developer
portal. See [Publishing a Stack](docs/developers/PUBLISHING.md).

## How do Flows get made?

Ask an Assistant in chat to build a Flow from a plain-language description, or use
**+ New flow** on the Flows page and choose **Describe it** to fill the form for you.

## Can I copy a setup to another account?

You can export your workspace's connections, policies and workflows as a bundle with
`GET /api/workspaces/{workspace_id}/export`, and import a bundle into your workspace with
`POST /api/workspaces/{workspace_id}/import` and a body of `{"bundle": …}`. Add
`?dryRun=true` to preview, and `?sections=connections,policies,workflows` to choose what
to import. The bundle has no credential fields, so reconnect services after an import.

## What happens when a connection's authorization expires?

Reconnect the service from **Integrations**.

## Where can I see which connectors exist?

In **Integrations** in Conducktor.

## Who do I contact?

- Support: **support@conducktor.com**. Include the time of the problem and what you were
  doing.
- Building and publishing Stacks: **developers@conducktor.com**.
- Security issues: **security@conducktor.com** (see [SECURITY.md](SECURITY.md)).
