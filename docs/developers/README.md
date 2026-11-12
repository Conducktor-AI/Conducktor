# Developer Platform

Every Conducktor user has a workspace built around them, where their tools, connections
and Assistants work together. As a developer you can run your own code there as a
**Build**, bring Builds, Assistants, Flows and knowledge together in a **Stack** that
installs in one step, and submit a Stack for the **Stack library**, so that every workspace
can add it. Publishing is optional: a Stack is fully usable in your own workspace.

## The model

- **A Build is code that runs on Conducktor.** You upload a package (or Builder writes the
  code with you), and Conducktor builds a container image and runs it in your workspace.
  A Build can serve its own screen and expose HTTP operations.
- **A Build lives inside a Stack.** A new Build gets its own private Stack, unless you add
  it to a Stack you already have. A Stack can hold several Builds.
- **Every endpoint becomes a tool.** When a Build launches, Conducktor reads its OpenAPI
  document and registers each operation as a tool on the Stack's connector. Assistants in
  the workspace can call those tools, and so can AI clients such as Claude and ChatGPT
  through `ask_conducktor`.
- **A Stack installs together.** A Stack can bundle Builds with Assistants, Flows, the
  connectors it needs and knowledge documents. Installing it adds its Assistants, Flows
  and knowledge to the workspace, asks the user to connect its connectors, and starts its
  Builds.
- **A Stack can reach every workspace.** A Stack you write stays private to your
  workspace until you send it for review from the developer portal. Once it passes,
  Conducktor adds it to the Stack library as a third-party Stack. See
  [Publishing a Stack](PUBLISHING.md).
- **Identity is by id.** A Build and a Stack each have an immutable id (`build_id`,
  `template_id`). Names are labels and can repeat.

## Documentation

| Guide | What it covers |
|-------|----------------|
| [Getting started](GETTING_STARTED.md) | Upload a small FastAPI service, launch it, and see its operations become tools |
| [Package specification](APP_PACKAGE_SPEC.md) | What a package contains, how the runtime is chosen, upload and build limits |
| [Build and deploy](BUILD_AND_DEPLOY.md) | The endpoints to upload, launch, follow and operate a Build |
| [Runtime contract](APP_RUNTIME.md) | What a running Build gets: environment, config and secrets, data store, connectors, models, schedules and webhooks, network, resources |
| [Stack authoring](STACK_AUTHORING.md) | Describe a Stack in one manifest, `stack.yaml` |
| [Stack package](STACK_PACKAGE.md) | Ship a whole Stack in one ZIP: manifest, specialists, knowledge and Builds |
| [Publishing a Stack](PUBLISHING.md) | Send a Stack for review and have it added to the Stack library |
| [Platform APIs](PLATFORM_APIS.md) | Authentication, the MCP endpoint, inbound webhooks, API-Bridge |

Related: the [OAuth guide](../authentication/OAUTH_GUIDE.md), for AI clients signing in
to Conducktor and for MCP servers that users connect.

## What a Build gets

| Service | Details |
|---------|---------|
| **Runtime** | Container build, hosting in an isolated, workspace-scoped environment, launch, stop and restart |
| **Tools** | Each OpenAPI operation registered as a tool on the Stack's connector |
| **Data store** | A per-Build key/value store with its own token and an SDK client |
| **Database** | Its own SQL database, reached through the platform ([Runtime contract § Your own database](APP_RUNTIME.md#your-own-database)) |
| **Long-running jobs** | Work that runs in steps beyond one request, with progress and checkpoints ([Runtime contract § Long-running jobs](APP_RUNTIME.md#long-running-jobs)) |
| **Files** | Read and write the Build's own Drive folder, and the folders the user grants |
| **Connectors** | Call the workspace connectors the user grants, without holding their credentials |
| **Models** | Ask a Claude model, with text, images or PDFs, your own tools, and streaming, paid from the workspace's Credits, without holding a model key |
| **Scheduled jobs** | Declare a job; Conducktor calls the Build on the schedule the user turns on, starting it if it is stopped |
| **Inbound webhooks** | Declare a webhook; Conducktor checks each delivery's signature and passes the fields you name to the Build |

The [Runtime contract](APP_RUNTIME.md) has the details, and lists what is not available
yet.

## Getting code in

| Source | How |
|--------|-----|
| Builder | Describe what you want in Conducktor; Builder writes the code with you and launches it. |
| ZIP | In **Stacks**: **New Stack**, then **Import**, then **ZIP file**. Source code becomes a Build ([Getting started](GETTING_STARTED.md)); a ZIP with `stack.yaml` at its root becomes a Stack ([Stack package](STACK_PACKAGE.md)) |
| Drive | A ZIP document in Drive, or a top-level folder of Drive's Builds collection ([Build and deploy](BUILD_AND_DEPLOY.md#get-the-source-in)) |
| GitHub | A repository, from **Stacks**: **New Stack**, then **Import**, then **GitHub** |
| Hugging Face | A Space, by `owner/name` ([Build and deploy](BUILD_AND_DEPLOY.md#get-the-source-in)) |

## The lifecycle at a glance

```
package   an HTTP service with /openapi.json and a health route
upload    a ZIP, from Stacks → New Stack → Import     a Build, in its own Stack
launch    image built, container running
tools     each OpenAPI operation registered on the Stack's connector
called    by Assistants, Flows and AI clients (through ask_conducktor)
package   stack.yaml plus Builds and knowledge, in one ZIP (a Stack package)
publish   Send for review in the developer portal
```

## Support

- Building and publishing Stacks: **developers@conducktor.com**
- Security: [SECURITY.md](../../SECURITY.md)
- Privacy: [PRIVACY.md](../../PRIVACY.md)
- Your account or workspace: **support@conducktor.com**
