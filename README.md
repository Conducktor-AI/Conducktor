# Conducktor Documentation

**Your workspace. Built around you.** Conducktor is one place to build, connect and run
your own tools. Connect your services once, then use them in your workspace or through the
AI assistant you prefer, such as Claude or ChatGPT. Create with Builder, or bring work you
have already started, and Conducktor hosts and runs it in your workspace, on the web and in
Conducktor's mobile apps. Nothing runs on your own computer, and credentials are stored
encrypted in the workspace's Vault and used on its behalf.

What you build lives in a **Stack**: the Builds, connections, tools and Assistants needed
for a purpose. Use a Stack privately in your workspace, or submit it for the **Stack
library**, where every workspace can add it. Publishing to the Stack library is optional.

These docs are the technical reference: how the pieces fit, and how to build on them,
publish to them, and reach them from other AI clients.

## The pieces

- An **Assistant** does work for the user in chat and in Flows.
- A **Flow** is a set of repeatable steps that runs on demand, on a schedule, or when an
  outside service sends a webhook.
- A **Build** is code that runs on Conducktor. It can serve its own screen, and each of its
  HTTP operations becomes a tool.
- A **Stack** brings together the Builds, Assistants, Flows, knowledge and connections
  needed for a purpose, and installs them together. Users install Stacks from the Stack library
  or make their own.
- A **connector** is anything with endpoints: a service the user connects (an MCP server,
  or a REST API bridged from its OpenAPI spec), and every Build. A Stack's Builds share one
  connector for that Stack. Every tool call goes through Conducktor's governance checks
  (permissions, rate limits, argument validation) and is audit-logged.

From outside, an AI client (Claude, ChatGPT, Cursor, or your own agent) connects to
`https://mcp.conducktor.com/mcp`. It sees Conducktor's gateway tools, chiefly
`ask_conducktor`, which takes a plain-language request and does the routing, tool calls
and follow-up questions inside Conducktor. The individual connector tools stay inside
Conducktor. See [Platform APIs § MCP endpoint](docs/developers/PLATFORM_APIS.md#mcp-endpoint).

## Build and publish a Stack

1. Start with the [Stack examples](stack-examples/): each is one `stack.yaml` you can upload
   to your workspace and install in a minute.
2. Change it, following [Stack authoring](docs/developers/STACK_AUTHORING.md), and add
   your own code as Builds if the Stack needs it. A [Stack package](docs/developers/STACK_PACKAGE.md)
   carries the manifest, your Builds and your knowledge in one ZIP.
3. Use it in your own workspace for as long as you like.
4. When it is ready for everyone, send it for review from the developer portal
   ([Publishing a Stack](docs/developers/PUBLISHING.md)).

Pricing and payouts for published Stacks: <https://www.conducktor.com/developers>.

## Who these docs are for

- **Developers** who run code on Conducktor as Builds, package Builds, Assistants and
  knowledge into a Stack, and publish Stacks to the Stack library.
- **Teams connecting an AI client or agent** to Conducktor over MCP.
- **Service providers** whose MCP server or API users connect to Conducktor.

## Documentation

| Document | What it covers |
|---|---|
| [Developer platform](docs/developers/README.md) | How Builds, Stacks and connectors fit together, and where to start |
| [Getting started](docs/developers/GETTING_STARTED.md) | Upload a small HTTP service, launch it, and see its operations become tools |
| [Package specification](docs/developers/APP_PACKAGE_SPEC.md) | What an uploaded package contains, how the runtime is chosen, upload and build limits |
| [Build and deploy](docs/developers/BUILD_AND_DEPLOY.md) | The upload, launch and lifecycle endpoints for a Build |
| [Runtime contract](docs/developers/APP_RUNTIME.md) | What a running Build gets: environment, config and secrets, data store, connectors, models, schedules and webhooks, network |
| [Stack authoring](docs/developers/STACK_AUTHORING.md) | Describe a Stack in one manifest, `stack.yaml` |
| [Stack package](docs/developers/STACK_PACKAGE.md) | Ship a whole Stack in one ZIP: manifest, specialists, knowledge and Builds |
| [Publishing a Stack](docs/developers/PUBLISHING.md) | Send a Stack for review and have it added to the Stack library |
| [Platform APIs](docs/developers/PLATFORM_APIS.md) | Authentication, the MCP endpoint, inbound webhooks, API-Bridge |
| [OAuth guide](docs/authentication/OAUTH_GUIDE.md) | OAuth for MCP servers that users connect to Conducktor, and Conducktor's own OAuth endpoints |
| [ChatGPT](docs/integrations/CHATGPT.md) | Connect ChatGPT to Conducktor |
| [FAQ](FAQ_CONSUMERS.md) | Short answers for teams connecting clients and services |
| [Security policy](SECURITY.md) | Security properties and how to report a vulnerability |
| [Privacy](PRIVACY.md) | Where the privacy policy is published |

How to connect each MCP client step by step: <https://www.conducktor.com/docs>.

## Help and contributions

Questions and help: [SUPPORT.md](SUPPORT.md). Fixes and new examples are welcome; read
[CONTRIBUTING.md](CONTRIBUTING.md) first.

## Reporting a security issue

Do not open a public issue. Email **security@conducktor.com**; see [SECURITY.md](SECURITY.md)
for what to include and what happens next.

## License

This documentation is licensed under [Creative Commons Attribution 4.0 International (CC BY 4.0)](LICENSE).
