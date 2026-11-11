# Conducktor Documentation

**Conducktor is your personal AI workspace.** It connects to everything you use, builds
what you need, and works for you, privately. The workspace is yours: your credentials are
stored encrypted in its Vault and used on your behalf.

These docs are the technical reference: how the pieces fit, and how to build on them,
connect to them, and reach them from other AI clients.

Your workspace is made of **Assistants** and **Stacks**.

- An **Assistant** does work for you in chat and in Flows.
- A **Stack** is everything for one job, installed together: the **Builds** that do the
  work (code that runs on Conducktor; each Build can serve its own surface), the Assistants and Flows
  that run it, and the connectors it needs. You install a Stack from the library or make
  your own with Builder.

Everything with endpoints is a **connector**: a service you connect (an MCP server, or a
REST API bridged from its OpenAPI spec), and every Build, whose HTTP operations become
tools. A Stack's Builds share one connector for that Stack. Tool calls go through
Conducktor's governance checks (permissions, rate limits, argument validation) and is
audit-logged.

From outside, an MCP client (Claude, ChatGPT, Cursor, or your own agent) connects to
`https://mcp.conducktor.com/mcp` and, by default, sees two tools: `ask_conducktor`, which takes a
plain-language request and does the routing, tool calls and follow-up questions inside
Conducktor, and `send_to_conducktor`, which sends a file. The individual connector tools
stay inside Conducktor.

## Build a Stack

Start with the [Stack examples](stack-examples/): each is one `stack.yaml` you can post to
your workspace and install in a minute. Copy one, change it, and follow
[Stack authoring](docs/developers/STACK_AUTHORING.md) for everything a manifest can hold.
Want to publish Stacks for other people? See <https://www.conducktor.com/developers>.

Contributions are welcome: read [CONTRIBUTING.md](CONTRIBUTING.md), and get help in
[SUPPORT.md](SUPPORT.md).

## Who these docs are for

- **Developers** who run code on Conducktor as Builds, or package Builds, Assistants and
  knowledge into a Stack.
- **Teams connecting an AI client or agent** to Conducktor over MCP.
- **Service providers** whose MCP server or API users connect to Conducktor.

## Documentation

| Document | What it covers |
|---|---|
| [Developer platform](docs/developers/README.md) | How Builds, Stacks and connectors fit together, and where to start |
| [Getting started](docs/developers/GETTING_STARTED.md) | Upload a small HTTP service, launch it, and see its operations become tools |
| [Package specification](docs/developers/APP_PACKAGE_SPEC.md) | What an uploaded package contains, how the runtime is chosen, upload and build limits |
| [Build and deploy](docs/developers/BUILD_AND_DEPLOY.md) | The upload, launch and lifecycle endpoints for a Build |
| [Runtime contract](docs/developers/APP_RUNTIME.md) | What a running Build gets: environment, config and secrets, data store, platform services, network |
| [Stack authoring](docs/developers/STACK_AUTHORING.md) | Describe a Stack in one manifest and add it to your workspace |
| [Platform APIs](docs/developers/PLATFORM_APIS.md) | Authentication, the MCP endpoint, Flow triggers, API-Bridge |
| [OAuth guide](docs/authentication/OAUTH_GUIDE.md) | OAuth for MCP servers that users connect to Conducktor, and Conducktor's own OAuth endpoints |
| [ChatGPT](docs/integrations/CHATGPT.md) | Connect ChatGPT to Conducktor |
| [Linear](docs/integrations/LINEAR.md) | Connect Linear as a connector |
| [Redis](docs/integrations/REDIS.md) | Connect a Redis database as a connector |
| [FAQ](FAQ_CONSUMERS.md) | Short answers for teams connecting clients and services |
| [Security policy](SECURITY.md) | Security properties and how to report a vulnerability |
| [Privacy](PRIVACY.md) | Where the privacy policy is published |

How to connect each MCP client step by step: <https://www.conducktor.com/docs>.

## Reporting a security issue

Do not open a public issue. Email **security@conducktor.com**; see [SECURITY.md](SECURITY.md)
for what to include and what happens next.

## License

This documentation is licensed under [Creative Commons Attribution 4.0 International (CC BY 4.0)](LICENSE).
