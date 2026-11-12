# Platform APIs

> **Base URL:** `https://api.conducktor.com`

REST endpoints live under `/api/...`; there is no `/v1` segment. The MCP endpoint is
`/mcp`.

There are two kinds of caller, with different credentials:

- **You**, calling the APIs on this page with your user-session token or an OAuth token.
- **A running Build**, calling the data store and platform services with its own
  `CONDUCKTOR_APP_TOKEN`. That is covered in the [Runtime contract](APP_RUNTIME.md).

## Authentication

| Method | Use | Status |
|--------|-----|--------|
| User-session bearer token (JWT) | Calls to the workspace REST APIs: `Authorization: Bearer <token>`. A session cookie also works. | Available |
| OAuth (authorization code with PKCE) | MCP clients and agents acting for a user | Available |
| Dynamic Client Registration (RFC 7591) | Clients that register themselves | Available |
| Workspace API key | A long-lived server-to-server key | Not available. Connector API keys you enter on a connection are credentials for that service, not for Conducktor. |

OAuth endpoints:

| Endpoint | Purpose |
|----------|---------|
| `POST /api/oauth/register` | Dynamic Client Registration |
| `GET /api/oauth/authorize` | Authorization code flow (PKCE method `S256`) |
| `POST /api/oauth/token` | Code exchange, refresh, client credentials, device code |
| `POST /api/oauth/revoke` | Token revocation (RFC 7009) |
| `POST /api/oauth/introspect` | Token introspection (RFC 7662) |
| `POST /api/oauth/device/authorize` | Device authorization (RFC 8628) |
| `GET /.well-known/oauth-authorization-server` | Server metadata (RFC 8414), also at `/mcp/.well-known/oauth-authorization-server` |

Details and examples: [OAuth guide](../authentication/OAUTH_GUIDE.md#part-1-connecting-to-conducktor).

**Who may call what.** Each account owns one workspace, and you are its administrator.
Creating a Build (upload or import) requires a workspace administrator.

## MCP endpoint

MCP clients connect to:

```
https://mcp.conducktor.com/mcp
```

(`https://api.conducktor.com/mcp` serves the same endpoint.)

| Endpoint | Purpose |
|----------|---------|
| `GET`/`POST /mcp/` | MCP Streamable HTTP transport (JSON-RPC 2.0) |
| `GET /mcp/sse` | Legacy SSE transport |
| `GET /mcp/tools/list` | The tools this client is shown |
| `POST /mcp/tools/call` | Call one of them |
| `GET /mcp/manifest` | Server manifest |
| `GET /mcp/health` | Health check |
| `DELETE /mcp/session` | End an MCP session |

A client is shown Conducktor's gateway tools, not the individual connector tools:

- `ask_conducktor` takes a `command` in plain language. Conducktor works out which
  connector tools to use (including your Builds' tools), runs them through its governance
  checks, and returns the result or a follow-up question.
- `send_to_conducktor` sends a file: Conducktor scans it, saves it to Drive and returns a
  receipt.
- `conducktor_vault` asks whether the client may sign in to a website with one of the
  user's saved logins and, if not, returns a Conducktor link where the user can allow it
  (or, with `revoke`, end it). It never returns passwords, cookies or tokens; consent
  happens in the user's own Conducktor session.

How to connect each client: <https://www.conducktor.com/docs>.

### Attach an MCP server you host

If your service already speaks MCP over HTTPS, attach it to your workspace as a
connection with `POST /api/connections/`, with the auth it needs (OAuth, API key, or
Basic). Nothing is deployed on Conducktor; calls are proxied to your server.

## Data store (summary)

A running Build's durable key/value store. Full details, including the SDK, are in the
[Runtime contract](APP_RUNTIME.md#data-store--your-durable-state).

- Base path: `/api/app-data`
- Auth: the Build's own `CONDUCKTOR_APP_TOKEN`. The token fixes the scope (workspace and
  Build), so one Build can never read another's keys.

| Endpoint | Purpose |
|----------|---------|
| `GET /api/app-data?prefix=<p>` | List keys → `{ "keys": [...], "count": n }` |
| `GET /api/app-data/{key}` | Read → `{ "key": "...", "value": ... }` |
| `PUT /api/app-data/{key}` | Write `{ "value": <any JSON> }` |
| `DELETE /api/app-data/{key}` | Delete → `204` |

A key is one URL path segment of at most 256 characters, so it cannot contain `/`, not even
percent-encoded as `%2F` (such a request gets `404` and nothing is stored). Use another
separator, such as `:`.

Per Build: values up to 256 KB (`413` above that), 1,000 keys (`409` above that), 120
requests per 60 seconds (`429` above that). Load on start, work in memory, and write back
periodically, rather than writing on every request.

## Inbound webhooks

An outside service (a payment provider, a code host, your own system) can start a
Conducktor **Flow** by sending it a webhook.

1. In Conducktor, create a webhook under **Connectors** (Data Source, then Webhook). Choose
   a provider format where one is offered, or the Conducktor standard format below.
   Conducktor shows the webhook's address and its signing secret **once**; enter them in
   the sending service. The secret is kept in the workspace's Vault and cannot be read
   again.
2. Ask Conducktor to start a Flow from that webhook. A Build can also declare the Flow for
   you (see [Runtime contract § Scheduled jobs and webhooks](APP_RUNTIME.md#scheduled-jobs-and-webhooks)).

No session or API key is involved: the signature is the credential.

**The Conducktor standard format.** Send a `POST` whose body is a JSON object. Take the
current Unix time in seconds as `t`, compute the HMAC-SHA256 of `t`, a dot and the raw
body with the webhook's secret, and send `X-Event-Signature: t=<t>,v1=<hex digest>`. A
delivery signed more than 5 minutes ago is refused.

```bash
BODY='{"event":"order.created","id":"1234"}'
T=$(date +%s)
SIG=$(printf '%s.%s' "$T" "$BODY" | openssl dgst -sha256 -hmac "$WEBHOOK_SECRET" -hex | sed 's/^.*= //')
curl -X POST "$WEBHOOK_ADDRESS" \
  -H "Content-Type: application/json" \
  -H "X-Event-Signature: t=$T,v1=$SIG" \
  --data "$BODY"
```

A bad or stale signature answers `401`; a body that is not a JSON object answers `400`.

Flow trigger addresses made before webhooks moved to **Connectors**
(`POST /api/webhooks/flow-trigger/{trigger_id}`) keep working with the same format.

## API-Bridge (OpenAPI to tools)

If your service has a REST API with a spec, API-Bridge turns its operations into tools, with
nothing to deploy.

| Endpoint | Purpose |
|----------|---------|
| `POST /api/api-bridge/validate` | Validate a spec before importing it |
| `POST /api/api-bridge/create` | Create a bridge from an inline spec |
| `POST /api/api-bridge/upload` | Upload a spec file: OpenAPI 3.0, OpenAPI 3.1 or Swagger 2.0, JSON or YAML |
| `POST /api/api-bridge/import-url` | Import a spec from a URL (several at once: `POST /api/api-bridge/bulk-import-urls`) |
| `POST /api/api-bridge/import-wsdl` | Import a SOAP WSDL |
| `GET /api/api-bridge/{api_spec_id}/tools` | List the generated tools |
| `POST /api/api-bridge/{api_spec_id}/configure-auth` | Set how the bridge authenticates to your API |
| `POST /api/api-bridge/{api_spec_id}/tools/{tool_name}/execute` | Run one tool as a test |
| `POST /api/api-bridge/{api_spec_id}/install-as-connection` | Attach the bridge to your workspace as a connection |
| `GET`/`POST /api/api-bridge/{api_spec_id}/mcp` | The bridge's own MCP endpoint |

Calls through a bridge are sent to your REST API with the auth you configured.

## Which path to use

| You have | Use |
|----------|-----|
| Code you want to run on Conducktor | A Build: [Package specification](APP_PACKAGE_SPEC.md) and [Build and deploy](BUILD_AND_DEPLOY.md) |
| Builds, Assistants and knowledge to ship together | A Stack: [Stack package](STACK_PACKAGE.md) |
| An MCP server you host | [Attach it as a connection](#attach-an-mcp-server-you-host) |
| A REST API with an OpenAPI spec | [API-Bridge](#api-bridge-openapi-to-tools) |
| Events that should start automations | [Inbound webhooks](#inbound-webhooks) |
| A Stack you want every workspace to be able to install | [Publishing a Stack](PUBLISHING.md) |
| An AI client or agent that should use Conducktor | The [MCP endpoint](#mcp-endpoint) and the [OAuth guide](../authentication/OAUTH_GUIDE.md) |
