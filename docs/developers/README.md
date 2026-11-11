# Developer Platform

You can run your own code on Conducktor as a **Build**, and package Builds, Assistants,
Flows and knowledge into a **Stack** that installs in one step.

## The model

- **A Build is code that runs on Conducktor.** You upload a package (or Builder writes the
  code with you), Conducktor builds a container image and launches it in your workspace.
  A Build can serve its own surface (a web page) and can expose HTTP operations.
- **A Build lives inside a Stack.** A new Build gets its own private Stack, unless you add
  it to a Stack you already built. A Stack can hold several Builds, each with its own
  surface.
- **Everything with endpoints is a connector.** When a Build launches, Conducktor reads its
  OpenAPI document and registers each operation as a tool. A Stack has one connector, and
  it carries the tools of every Build in the Stack. Assistants in the workspace, and MCP
  clients through `ask_conducktor`, can then call them.
- **A Stack installs together.** A Stack can bundle Builds with Assistants, Flows, the
  connectors it needs and knowledge documents. Installing it adds its Assistants, Flows
  and knowledge to the workspace, asks the user to connect its connectors, and starts its
  Builds.
- **Identity is by id.** A Build and a Stack each have an immutable id (`build_id`,
  `template_id`); refer to them by id. Names are labels and can repeat.

## Documentation

| Guide | What it covers |
|-------|----------------|
| [Getting started](GETTING_STARTED.md) | Upload a small FastAPI service, launch it, and see its operations become tools |
| [Package specification](APP_PACKAGE_SPEC.md) | What a package contains, how the runtime is chosen, upload and build limits |
| [Build and deploy](BUILD_AND_DEPLOY.md) | The endpoints to upload, launch, follow and operate a Build |
| [Runtime contract](APP_RUNTIME.md) | What a running Build gets: environment, config and secrets, data store, platform services, network, resources |
| [Stack authoring](STACK_AUTHORING.md) | Describe a Stack in one manifest and add it to your workspace |
| [Platform APIs](PLATFORM_APIS.md) | Authentication, the MCP endpoint, Flow triggers, API-Bridge |

Related: the [OAuth guide](../authentication/OAUTH_GUIDE.md), for MCP clients signing in to
Conducktor and for MCP servers that users connect.

## What a Build gets

| Service | Status |
|---------|--------|
| **Runtime**: container build, hosting in an isolated workspace-scoped environment, launch, stop and restart | Available |
| **Tools**: OpenAPI operations registered as tools on the Stack's connector | Available |
| **Data store**: a per-Build key/value store with its own token and an SDK client | Available |
| **Files**: read and write the Build's own folder, and the Drive folders the user grants | Available |
| **Connectors**: call the workspace connectors the user grants, without holding their credentials | Available |
| **Event ingress**: HMAC-signed Flow triggers your external systems can call | Available (calls a Flow, not a Build) |
| **Model access**: the platform's own model as a service | Roadmap; a workspace's own Anthropic key can be injected |
| **Inbound webhooks to a Build** | Roadmap |

The [Runtime contract](APP_RUNTIME.md) has the details and the full list of what is not
available yet.

## Getting code in

| Source | How |
|--------|-----|
| Builder | Describe what you want in Conducktor; Builder writes the code with you and launches it. |
| ZIP package | `POST /api/builder/builds/upload` ([Getting started](GETTING_STARTED.md)) |
| Drive | A ZIP document in Drive (`POST /api/builder/builds/from-drive/{document_id}?workspace_id=<id>`), or a top-level folder of Drive's Builds collection (`POST /api/builder/builds/from-drive-folder/{folder_id}?workspace_id=<id>`) |
| Hugging Face | A Space, by `owner/name` (`POST /api/builder/builds/from-huggingface?space_id=owner/name&workspace_id=<id>`) |

## The lifecycle at a glance

```
package (an HTTP service with /openapi.json and a health route)
  → upload   POST /api/builder/builds/upload               → a Build, in its own Stack
  → launch   POST /api/builder/builds/{build_id}/launch    → image built, container running
  → tools    each OpenAPI operation registered on the Stack's connector
  → called   by Assistants, Flows and MCP clients (through ask_conducktor)
```

## Support

- Security: [SECURITY.md](../../SECURITY.md)
- Privacy: [PRIVACY.md](../../PRIVACY.md)
- Everything else: support@conducktor.com
