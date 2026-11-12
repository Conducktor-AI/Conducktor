# Runtime Contract

> **Scope:** what a running Build gets from Conducktor (environment, credentials, the SDK,
> and the services it can call) and what it must provide.

A launched Build runs as a container in an isolated, workspace-scoped environment.

The generated tables on this page come straight from the platform's contract. They call
the running Build "your app" and its key/value store the "App Data Store"; this page calls
them the Build and the data store.

## Environment variables

### Which launch path a Build runs on

Builds run on the **default launch path**. The platform can also run them on an **older
bundle path**. Where the two differ, it is noted below; everything else on this page is
the same for both.

### What the platform injects

On the default path a launched Build starts with this environment:

<!-- platform-contract:injected-env -->

| Variable | Meaning |
|----------|---------|
| `WORKSPACE_ID` | The workspace your app is deployed into |
| `CONDUCKTOR_APP_TOKEN` | Your app's bearer token for OS services: the App Data Store, and each platform service the workspace allows your app |
| `CONDUCKTOR_APP_DATA_URL` | Base URL of the App Data Store (`…/api/app-data`) |
| `CONDUCKTOR_APP_SERVICES_URL` | Base URL of the platform services (`…/api/app-services`); call them with `CONDUCKTOR_APP_TOKEN`. A call is `POST {base}/{service}/{tool}` with the tool's full name, e.g. `{base}/files/files.list`, never `{base}/files.list`; in Python, `conducktor_app.services` (and in a scaffolded Next.js app, `lib/platform-services.js`) builds it for you |
| `ANTHROPIC_API_KEY` | Only when your workspace has its **own** Anthropic key (a stored BYOK key or its own active Anthropic connection); never the platform's key. A value you set yourself through the config endpoint wins |
| `INSTANCE_ID` | Platform-internal; do not use. The platform writes it after your config, so a value you set is replaced |
| `INSTANCE_TOKEN` | Platform-internal; do not use. The platform writes it after your config, so a value you set is replaced |
| `CONDUCKTOR_INTERNAL_URL` | Platform-internal; do not use. The platform writes it after your config, so a value you set is replaced |
| `CONDUCKTOR_API_URL` | Base URL of the Conducktor API. The platform writes it after your config, so a value you set is replaced |
| `INSTANCE_TOKEN_REF` | Platform-internal; do not use. A reserved key: refused at `PUT` |
| `CONDUCKTOR_DATA_DIR` | Only when the Build declares a disk (`environment.filesystem`): the folder whose files survive stops, restarts, relaunches and new versions. The platform writes it after your config, so a value you set is replaced |

<!-- /platform-contract:injected-env -->

Each runtime also sets variables of its own, for example `PYTHONUNBUFFERED=1` for the
Python runtimes, and `NODE_ENV`, <!-- platform-contract:nextjs-port -->`PORT=3000`<!-- /platform-contract:nextjs-port --> and `HOSTNAME` for `nextjs`. Your
own config overrides these, **except reserved keys**: `PORT` is reserved (see below), so
`nextjs`'s <!-- platform-contract:nextjs-port -->`PORT=3000`<!-- /platform-contract:nextjs-port --> cannot be changed.

On the default path **`APP_SLUG`, `BUILD_ID`, `RUNTIME` and `PORT` are not injected**
(apart from the <!-- platform-contract:nextjs-port -->`PORT=3000`<!-- /platform-contract:nextjs-port --> that `nextjs` sets for itself). Do not branch on
`RUNTIME`. Take your port from the runtime's fixed start command or, for the `dockerfile`
runtime, from your `EXPOSE`; behind the shim, `PORT` is set to that port for your process
(see the [Package specification](APP_PACKAGE_SPEC.md#the-dockerfile-runtime)).

On the older bundle path the platform injects <!-- platform-contract:injected-env-bundle -->`WORKSPACE_ID`, `CONDUCKTOR_APP_TOKEN`, `CONDUCKTOR_APP_DATA_URL`, `CONDUCKTOR_APP_SERVICES_URL`, `INSTANCE_TOKEN_REF`, `APP_SLUG`, `BUILD_ID`, `PORT` and `RUNTIME`<!-- /platform-contract:injected-env-bundle -->
(`PORT` is the serving port; `RUNTIME` is one of `fastapi`, `streamlit`, `gradio`,
`nextjs`, `vite_static`, `static_html`, `dockerfile`).

### Your own config and secrets

Set your own variables and secrets on the Build:

```bash
curl -X PUT "https://api.conducktor.com/api/builder/builds/$BUILD_ID/config?workspace_id=$WORKSPACE_ID" \
  -H "Authorization: Bearer $CONDUCKTOR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "entries": [
      {"key": "LOG_LEVEL", "kind": "env", "isSecret": false, "value": "info"},
      {"key": "STRIPE_API_KEY", "kind": "env", "isSecret": true, "value": "sk_live_…"}
    ]
  }'
```

They are resolved at launch and delivered to the container. Read them back with
`GET /api/builder/builds/{build_id}/config?workspace_id=<id>` (secret values come back
masked, as `valuePreview`), and remove one with
`DELETE /api/builder/builds/{build_id}/config/{key}?workspace_id=<id>`. The `workspace_id`
query parameter is **required** on all three. Config belongs to the Build's project, so
later versions of the Build keep it.

🔴 **`PUT` replaces the whole list.** Any stored entry you leave out is **deleted**, a
secret's stored value with it, and secret values cannot be read back. To change one
entry, `GET` the list and `PUT` every entry back. For an entry whose value you are not
changing, omit `value` (or send `null`): the stored value is kept. Do not send the masked
`valuePreview` back as the value; it would be stored as the new value. A new entry needs a
value.

Each entry has a `kind`:

- `env`: <!-- platform-contract:config-kind-env -->delivered to the running app as an environment variable<!-- /platform-contract:config-kind-env -->. The key must be a valid environment variable name and may
  not be one of <!-- platform-contract:config-reserved-env-keys -->`WORKSPACE_ID`, `APP_SLUG`, `BUILD_ID`, `INSTANCE_TOKEN_REF`, `PORT`, `RUNTIME`, `CONDUCKTOR_APP_TOKEN`, `CONDUCKTOR_APP_DATA_URL` or `CONDUCKTOR_APP_SERVICES_URL`<!-- /platform-contract:config-reserved-env-keys -->
  (refused with `422`).
- `file`: <!-- platform-contract:config-kind-file -->**not supported on the default launch path**; with the workload runtime switched off it is mounted read-only at its absolute `targetPath`<!-- /platform-contract:config-kind-file -->. On the default path, a launch of a Build that has any
  `file` entry is refused before any running version is touched (it stays up), and the
  launch answers `502` with `Provisioning failed: config_files are not yet supported on the
  workload runtime …`. Either way the entry is checked when you `PUT` it: the path may not
  be `/` or a first-level path, may not sit under `/proc`, `/sys`, `/dev` or
  `/var/run/secrets`, may not collide with `/tmp`, `/var/cache/nginx` or `/var/run`, and
  may not nest with another entry's path.
- `build`: <!-- platform-contract:config-kind-build -->a build-time secret for your own Dockerfile (`RUN --mount=type=secret,id=<KEY>`); it **never** reaches the running app<!-- /platform-contract:config-kind-build -->. It must be a secret (see
  [Build secrets](APP_PACKAGE_SPEC.md#build-secrets)).

Limits, checked on every `PUT` (a violation is `422`): <!-- platform-contract:config-limits -->at most **32 entries** per app, **64 KiB** per value, and **512 KiB** each summed across the `env`, `file` and `build` values<!-- /platform-contract:config-limits -->. Each `PUT`
counts only the values sent in that `PUT` (an entry sent without a value counts zero), so
stored totals can grow past <!-- platform-contract:config-total -->512 KiB<!-- /platform-contract:config-total --> over several `PUT`s. The stored `env`
and `file` totals are checked at launch, and a launch over either is refused. The stored
`build` total is not checked at launch, so for `build` the <!-- platform-contract:config-total -->512 KiB<!-- /platform-contract:config-total --> bound is
per `PUT` only.

The launch request takes no environment variables, and there is no manifest to declare
them in; the config endpoint is the way. For state your Build writes while it runs, use the
data store. **Never put secrets in the package.**

### Settings an installer fills

Your own config stays with your Build: when another workspace installs your Stack, its
copy gets **none** of your values. For a value each installer must give (their region,
their account id, a webhook signing secret), mark the `env` entry for installers with an
`installer` object on the same `PUT`:

```json
{"key": "REGION", "kind": "env", "isSecret": false, "value": "eu-west",
 "installer": {"purpose": "Your account's region", "required": true}}
```

- `purpose` (required, at most 200 characters) is what the installer reads beside the field.
- `required` makes a copy wait until the value is filled; the rest of the Stack installs
  and runs, and the copy starts as soon as it is saved.
- A secret also needs a `reason` (at most 200 characters) saying why your code must hold
  the value itself. Conducktor's review checks it. A key for an outside web API should not
  be a secret setting: ship it as a connector, so the key stays with the platform.

Only `env` entries can be marked (`422` otherwise). Because `PUT` replaces the whole list,
send `installer` back with every entry you keep; an entry sent without it is no longer
asked of installers. `GET` returns each entry's `installer` (or `null`).

When the Stack is packaged, its submission lists each marked key with its purpose,
`required`, whether it is a secret and its reason, never a value. Installers fill the
values on the Stack's page, where a secret is stored in the Vault and shown again only as
a masked preview. The copy receives them as environment variables. Saving a change
restarts the copies that use it right away, so read them at start-up; your app's data is
kept, since it belongs to the installation, not to the pod.

A workspace runs one copy of a Build. If two installed Stacks ship the same Build, the copy
keeps the settings of the Stack that started it, and the other Stack's page says its
settings could not be applied.

## The Build token

`CONDUCKTOR_APP_TOKEN` is a signed bearer token scoped to exactly one Build in one
workspace. The scope comes from the token, never from request parameters, so one Build
cannot read another Build's data, even in the same workspace.

The token is accepted by the **data store** (`/api/app-data`) and by the **platform
services** at `CONDUCKTOR_APP_SERVICES_URL` (`/api/app-services`), each only once the user
has allowed it for the Build. No other Conducktor API accepts it. Two of the platform
services:

- **Files.** A Build can always read and write its own folder, and reads or writes other
  Drive folders (or all of Drive) only at the level the user granted.
- **Connectors.** A Build calls the tools of a workspace connector the user granted it,
  read-only or read and write, and Conducktor adds that connector's credentials on its
  side (see [Calling the workspace's connectors](#calling-the-workspaces-connectors)).

## Data store — your durable state

The filesystem is temporary; the data store is not. It is a per-Build key/value store that
survives restarts, redeploys and scaling.

Send `Authorization: Bearer $CONDUCKTOR_APP_TOKEN`:

| Endpoint | Purpose |
|----------|---------|
| `GET $CONDUCKTOR_APP_DATA_URL?prefix=<p>` | List keys → `{ "keys": [...], "count": n }` |
| `GET $CONDUCKTOR_APP_DATA_URL/{key}` | Read → `{ "key": "...", "value": ... }` |
| `PUT $CONDUCKTOR_APP_DATA_URL/{key}` | Write `{ "value": <any JSON> }` |
| `DELETE $CONDUCKTOR_APP_DATA_URL/{key}` | Delete → `204` |

A key is one URL path segment of at most 256 characters, so it cannot contain `/`, not even
percent-encoded as `%2F` (such a request gets `404` and nothing is stored). Use another
separator, such as `:`.

**Quotas** (per Build): <!-- platform-contract:app-data-quotas -->values up to **256 KB** (`413` above), **1,000 keys** (`409` above), **120 requests per 60 s** (`429` above)<!-- /platform-contract:app-data-quotas -->.

### The SDK

Every launched Build gets the `conducktor_app` Python module (SDK 1.6.0) at the root of its
source. It wraps the data store with the recommended pattern (load on start, work in
memory, write back periodically), which keeps you well inside the rate limit:

```python
from conducktor_app import store

store.hydrate()                 # load all keys once, at start
value = store.get("cursor")     # read from memory
store.put("cursor", {"page": 3})
store.flush()                   # write changed keys (also call it on shutdown)
```

`store` offers `get`, `put`, `delete`, `list`, `hydrate` and `flush`. Builds in other
languages call the REST endpoints directly, in the same pattern; a scaffolded Next.js
Build has `lib/platform-services.js` for the platform services.

## Your own database

When a Build's records have a shape worth querying (customers, orders, bookings, anything
you filter, join or total), keep them in the Build's own SQL database, the platform
service `db`, instead of the key/value store. Every Build gets a Postgres database of its
own, with its own login, holding nothing but what the Build writes. The tables survive restarts, redeploys and relaunches, and are deleted with
the Build. The Build never holds a password: Conducktor connects for it.

| Endpoint | Purpose |
|----------|---------|
| `POST $CONDUCKTOR_APP_SERVICES_URL/db/db.query` | Read with `{ "arguments": { "sql", "params"? } }`, run read-only → `{ "result": { "rows": [{...}], "row_count", "truncated" } }` |
| `POST $CONDUCKTOR_APP_SERVICES_URL/db/db.execute` | Change with one statement (`CREATE TABLE`, `INSERT`, `UPDATE`, `DELETE`, `ALTER` …) in its own transaction → the same shape; `rows` holds what `RETURNING` returns |
| `POST $CONDUCKTOR_APP_SERVICES_URL/db/db.batch` | Run `{ "arguments": { "statements": [{ "sql", "params"? }, ...] } }` in ONE transaction: all take effect or none do → `{ "result": { "results": [...] } }` |

- **`sql`** is one statement with `$1`, `$2`, … placeholders. Name tables plainly: they
  live in the Build's own database. Two statements in one `sql` are refused; use `db.batch`.
  Every call runs in its own transaction, so `BEGIN`, `COMMIT`, `ROLLBACK` and `SAVEPOINT`
  are refused too.
- **`params`** are the placeholders' values as JSON, converted to each column's type: ISO
  text for dates and times (`"2026-11-11T09:00:00Z"`), an object or list for `json` and
  `jsonb`, base64 for `bytea`. Rows come back as objects keyed by column name, with
  `numeric` as text so no digit is lost.
- Create tables at start with `CREATE TABLE IF NOT EXISTS`, in one `db.batch`, so a new
  Build and a relaunched one both start ready.

```python
from conducktor_app import services

services.db_batch([
    {"sql": "CREATE TABLE IF NOT EXISTS orders (id serial PRIMARY KEY, customer text, "
            "total numeric(10,2), placed timestamptz DEFAULT now())"},
])
services.db_execute("INSERT INTO orders (customer, total) VALUES ($1, $2)", params=["Ada", 42.5])
rows = services.db_query(
    "SELECT customer, sum(total) AS spent FROM orders GROUP BY customer ORDER BY spent DESC"
)["rows"]
```

A shared Build may use its database: keep what a visitor saves under the
`X-Conducktor-Visitor` name. An installed copy of your Stack gets its own empty database in
the installing workspace; it never sees yours.

### Limits

| Limit | Value |
|-------|-------|
| Statements per call | 1 (`db.batch`: 50, in one transaction) |
| Time per statement | 10 seconds |
| Time per batch | 30 seconds, then the whole batch is undone |
| Rows per reply | 1,000 (`truncated` is true when there were more; a batch's statements share it) |
| Reply size | 4 MB |
| Database size | 1 GB in all; past it only `DELETE`, `DROP` and `TRUNCATE` run |
| Calls per minute | 600 per workspace |

A Build may use only its own database and schema: no other database, no temporary tables,
no extensions.

### Errors

| Status | `code` | What it means |
|--------|--------|---------------|
| `400` | `sql_error` | Postgres refused the statement; the message starts with its SQLSTATE |
| `422` | `invalid_argument` | The SQL or a param is not valid; the message names which (`params[0]`, `statements[2]`) |
| `408` | `timeout` | The statement ran longer than 10 seconds |
| `429` | `busy` | Too many calls from this Build at once; retry |
| `507` | `database_full` | The database is at 1 GB; delete rows or drop tables, and the next write gives the space back |
| `403` | `service_off` / `not_allowed` | The database is turned off on the platform, or the workspace has switched it off for this Build |
| `503` | `unavailable` | The database could not be reached; try again shortly |

## Calling the workspace's connectors

A Build never holds a key for a service the workspace has connected. The user grants the
Build the connectors it may use, read-only or read and write. The Build calls them by
connection id; Conducktor adds the credentials, runs the call through the same checks as
any connector call, and returns the answer with any echo of those credentials replaced by
`[redacted]`.

| Endpoint | Purpose |
|----------|---------|
| `GET $CONDUCKTOR_APP_SERVICES_URL/connections` | The connectors the Build may call → `{ "result": { "connections": [{ "connection_id", "name", "connector", "level" }] } }` |
| `POST $CONDUCKTOR_APP_SERVICES_URL/connections/{connection_id}/{tool}` | Call one tool with `{ "arguments": {...} }` → `{ "result": { "is_error", "content", "structured" } }` |

A connector the Build was not granted answers `403` with code `not_granted`; a write tool
under a read grant answers `403` with code `read_only`. A tool on an MCP server, and a SOAP
operation, always needs a write grant: it is sent by name, and a server's own
`readOnlyHint` is not trusted to make it a read. A tool that reads or issues a credential
(an API key, a client secret, an access token, a password) answers `403` with code
`credential_tool` under any grant. An error from the remote service itself is a `200` with
`is_error: true`. In Python:

```python
from conducktor_app import services

services.granted_connectors()
services.call_connector(connection_id, "list_invoices", {"status": "open"})
```

### Connectors in an installed copy

When another workspace installs your Stack, its copy of your Build calls **the installing
workspace's** connections, never yours. Declare each connector the copy needs: tell the
Builder, for example *"installers' copies read Gmail with list_messages and get_message,
to summarise the inbox"*. A declaration is the connector, the most it may do (read unless
you say write), the tools it calls and a one-line reason. It is part of your Stack's
submission, so Conducktor's review sees it.

Nothing is granted at install. A member picks one of their own connections of that
connector on the Stack's page, the same one the Stack's assistants use, at read or (when
you declared it) read and write. The copy then reaches only the tools you declared: any
other answers `403` with code `not_declared`, and a connection nobody allowed answers
`not_granted`. Find what it may call with `services.granted_connectors()`, since its
connection ids are the installer's, and handle an empty list.

## Calling Claude models

A Build can ask a Claude model for an answer through the platform service `ai`. It never
holds a model key: Conducktor calls the model on its own key and runs the call through the
same checks as every platform service, plus the input guardrails chat uses.

| Endpoint | Purpose |
|----------|---------|
| `POST $CONDUCKTOR_APP_SERVICES_URL/ai/ai.complete` | Ask a model with `{ "arguments": { "messages": [...], "system"?, "model"?, "max_tokens"?, "temperature"?, "tools"?, "tool_choice"? } }` (a message's `content` is text or a list of text, image, PDF, tool call and tool result blocks) → `{ "result": { "text", "content", "model", "stop_reason", "usage": { "input_tokens", "output_tokens", "cache_read_tokens", "cache_write_tokens" } } }` |

- **`messages`** is the conversation so far, oldest first: `{ "role": "user" | "assistant",
  "content": "<text>" }`, starting and ending with the user's message. The platform keeps no
  history: the Build stores the conversation (the App Data Store is a good place) and sends
  it with each call.
- **`system`** is your instructions. Send the same text on every call: it is cached, so
  repeat calls cost less. In a conversation of three or more messages the history is cached
  too.
- **`model`** is one of the Claude models the platform offers; leave it out for the
  platform's default. **`max_tokens`** bounds the answer (1,024 by default).
- **Images and PDFs.** A message's `content` can also be a list of blocks, in the shape
  of Claude's Messages API: `{ "type": "text", "text": "..." }`, and in the user's
  messages `{ "type": "image", "source": { "type": "base64", "media_type": "image/jpeg",
  "data": "<base64>" } }` (JPEG, PNG, GIF or WebP) or `{ "type": "document", "source":
  { "type": "base64", "media_type": "application/pdf", "data": "<base64>" } }`. Use it to
  read a photo, a receipt or an invoice. An image or PDF is charged as the input tokens
  the model reports. A file whose bytes are not the type it names is refused.
- **Tools.** Send `tools` (each `{ "name", "description", "input_schema" }`, Claude's tool
  definition shape) and optionally `tool_choice` (`{ "type": "auto" }`, `{ "type": "any" }`
  or `{ "type": "tool", "name": "..." }`). The answer's `content` holds its text and any
  `tool_use` blocks. When `stop_reason` is `"tool_use"`, append `{ "role": "assistant",
  "content": <the answer's content> }` to the conversation, run each tool in your Build,
  send the results back as one user message whose content starts with
  `{ "type": "tool_result", "tool_use_id": "...", "content": "..." }` blocks (add
  `"is_error": true` for a tool that failed), and call again with the same tools. The
  platform never runs a Build's tools; every call of the loop is a separate, charged call.
- **Streaming.** `POST $CONDUCKTOR_APP_SERVICES_URL/ai/ai.complete/stream` takes the same
  body and answers with server-sent events while the model writes: `event: text` with
  `{ "text": "<the next piece>" }`, then one `event: done` with `{ "result": <the same
  answer ai.complete gives> }`. Use it for a chat screen that shows the reply as it is
  written (`services.ai_stream(...)` in Python, `aiStream(...)` in the Next.js scaffold,
  both yield the pieces). The text arrives a few sentences at a time, because each
  piece passes the same content screen as a whole answer before your Build sees it; if
  the screen stops a piece, the stream ends with `event: error` and `content_blocked`
  (what came before it stands). A call refused before any text answers with the same status
  and `code` as `ai.complete`; a failure after some text ends the stream with `event:
  error` and `{ "status", "code", "message" }`. Both SDK helpers raise `stream_incomplete`
  (502) if the connection ends before `done` or `error`, so a cut-off answer is never
  taken as whole. A streamed answer never calls a tool and takes no
  `tool_choice`: run the tool rounds with `ai.complete`, then stream the final answer with
  the same `tools` (a conversation that holds tool calls needs them). If your Build stops reading,
  the answer still finishes and is charged, so do not start a second call to retry a
  stream your own screen dropped.

```python
from conducktor_app import services

reply = services.ai_complete(
    [{"role": "user", "content": "Summarise this invoice: ..."}],
    system="You extract invoice fields and answer in one short paragraph.",
    max_tokens=400,
)
print(reply["text"])
```

```python
import base64

photo = base64.b64encode(open("receipt.jpg", "rb").read()).decode()
reply = services.ai_complete(
    [{"role": "user", "content": [
        {"type": "image", "source": {"type": "base64", "media_type": "image/jpeg", "data": photo}},
        {"type": "text", "text": "List the merchant, date and total as JSON."},
    ]}],
    max_tokens=300,
)
```

```python
from fastapi.responses import StreamingResponse

@app.post("/chat")
def chat(body: dict):
    return StreamingResponse(services.ai_stream(body["messages"], max_tokens=1000), media_type="text/plain")
```

```python
tools = [{
    "name": "find_orders",
    "description": "Orders for a customer email.",
    "input_schema": {"type": "object", "properties": {"email": {"type": "string"}}, "required": ["email"]},
}]
history = [{"role": "user", "content": "What did ana@example.com order last week?"}]
while True:
    reply = services.ai_complete(history, tools=tools, max_tokens=800)
    history.append({"role": "assistant", "content": reply["content"]})
    if reply["stop_reason"] != "tool_use":
        break
    history.append({"role": "user", "content": [
        {"type": "tool_result", "tool_use_id": b["id"], "content": run_my_tool(b["name"], b["input"])}
        for b in reply["content"] if b["type"] == "tool_use"
    ]})
print(reply["text"])
```

### Who pays

Every call is paid from the **workspace's Credits**, at the same markup as the workspace's
assistants, whatever the workspace's AI Source list says. A call never runs on a person's
own Claude subscription or own API key. The answer's `usage` reports the tokens the call
used.

When another workspace installs your Stack, its copy of your Build calls models the same
way, paid from **the installing workspace's** Credits and held to the daily limit that
workspace sets on the Stack's page. AI is **off** in an installed copy until a member of
the installing workspace turns on **Let this Stack's Builds use AI** on the Stack's page,
which shows the cost and the daily limit. Until then `ai.complete` answers
`403 not_allowed`, so handle that answer in your app. They can turn it off again at any
time.

### Limits

| Limit | Value |
|-------|-------|
| Messages per call | 200 |
| Characters per call (all messages and the system prompt) | 400,000 |
| Tools per call | 32 (100,000 characters together) |
| Images and PDFs per call | 20 |
| One image or PDF | 5 MB of base64 (a file of about 3.7 MB) |
| All images and PDFs in a call | 20 MB of base64 |
| Request body | 32 MB |
| System prompt | 100,000 characters |
| Answer length (`max_tokens`) | 8,192 tokens (1,024 by default) |
| Calls per minute | 60 per workspace |
| Time per call | 120 seconds |
| AI spend per Stack per day | A default set by Conducktor, unless the workspace sets another limit on the Stack's page (0 stops the Stack's AI calls). The Stack's page shows the limit, today's spend and the highest limit allowed |

The daily limit covers every Build in the Stack and resets at 00:00 UTC. A call that is
already running when the limit is reached still finishes, so a day's spend can go slightly
over it.

**Need more?** These limits apply to every Build. To raise one for your Stack, email
support@conducktor.com with the Stack, the limit you need, and why your Stack needs it (for example, the
expected calls per user per day). Increases are granted case by case. The Stack's daily
spending limit needs no request: the workspace changes it on the Stack's page.

### Errors

| Status | `code` | What it means |
|--------|--------|---------------|
| `422` | `invalid_argument` | A message, block, file, model or limit is not valid; the message says which |
| `413` | `too_large` | The request is over the character limit, or its images and PDFs are over theirs |
| `422` | `content_blocked` | The content policy blocked the request's text, or a streamed answer's next piece |
| `429` | `spending_limit_reached` | The Stack used its AI spending limit for today |
| `402` | `insufficient_credits` | The workspace's Credits or its own spending cap cannot cover the call |
| `409` | `refused` | The per-minute rate limit, or another platform check, refused the call |
| `403` | `service_off` / `not_allowed` | AI calls are turned off on the platform, or the workspace has not allowed this Build to use them |
| `503` | `unavailable` | The model call did not complete, or AI calls are switched off for now; try again shortly |

## Long-running jobs

A Build stops after a while without visitors, and one request into it has a time limit, so
work that takes longer (crawling a site, processing media or a large file, a long AI task)
runs as a **job** the platform drives, through the platform service `jobs`. Never run it as
a background thread or loop: nothing runs while the Build is stopped.

1. Give the work its own `POST` endpoint, without path parameters, whose request body
   declares `job_id`, `input`, `checkpoint` and `step`.
2. Start a job from your server code with `jobs.start`. It answers at once with the job.
3. The platform calls your endpoint step by step, starting the Build when it is stopped,
   each time with the job's `input`, the `checkpoint` the previous step returned (`null`
   the first time) and the step number. Each step does a bounded piece of the work and
   answers:
   - `{ "done": false, "checkpoint": <JSON>, "progress": "<text>" }` to continue;
   - `{ "done": true, "result": <JSON>, "progress": "<text>" }` when the job is finished;
   - `{ "error": "<text>" }` to fail the job.
4. Read a job with `jobs.get` or `jobs.list` (status `queued`, `running`, `succeeded`,
   `failed` or `cancelled`, the last progress line, the result or the error), and stop one
   with `jobs.cancel`.

| Endpoint | Purpose |
|----------|---------|
| `POST $CONDUCKTOR_APP_SERVICES_URL/jobs/jobs.start` | `{ "arguments": { "call": "POST /jobs/crawl", "input"?: {...}, "name"?: "..." } }` → `{ "result": { "job": {...} } }` |
| `POST $CONDUCKTOR_APP_SERVICES_URL/jobs/jobs.get` | `{ "arguments": { "job_id": "..." } }` → `{ "result": { "job": {...} } }` |
| `POST $CONDUCKTOR_APP_SERVICES_URL/jobs/jobs.list` | `{ "arguments": { "status"?, "limit"?, "offset"? } }` → `{ "result": { "jobs": [...], "next_offset" } }` |
| `POST $CONDUCKTOR_APP_SERVICES_URL/jobs/jobs.cancel` | `{ "arguments": { "job_id": "..." } }` → `{ "result": { "job": {...} } }` |

```python
from fastapi import FastAPI
from pydantic import BaseModel
from conducktor_app import services

app = FastAPI()

class Step(BaseModel):
    job_id: str
    input: dict | None = None
    checkpoint: dict | None = None
    step: int

@app.post("/crawl")
def start_crawl(body: dict):
    return services.jobs_start("POST /jobs/crawl", input={"site": body["site"]}, name="Crawl")

@app.post("/jobs/crawl")
def crawl_step(body: Step):
    page = (body.checkpoint or {}).get("page", 0)
    ...  # fetch and save a few pages, at most a few minutes of work
    if finished:
        return {"done": True, "result": {"pages": page}}
    return {"done": False, "checkpoint": {"page": page + 10}, "progress": f"{page + 10} pages"}
```

A step can run again with the same checkpoint (after a timeout or a platform restart), so
make every step safe to repeat. A job reaches only your Build's own endpoints, and a shared
Build may run jobs.

### Limits

| Limit | Value |
|-------|-------|
| Time per step | 5 minutes |
| Time per job, from its first step | 24 hours |
| Steps per job | 10,000 |
| Failed steps in a row before the job fails | 5 (retried after 30 s, 1, 2 and 4 minutes) |
| Unfinished jobs per Build | 20; one step of a Build's jobs runs at a time |
| `input` | 64 KB |
| `checkpoint` and `result` | 256 KB each: save larger output in Drive or the App Data Store |
| Finished jobs kept | 7 days |

### Errors

| Status | `code` | What it means |
|--------|--------|---------------|
| `422` | `invalid_argument` | `call` is not a `POST` path, or its endpoint does not declare `checkpoint` |
| `404` | `not_found` | `call` is not one of this Build's endpoints, or no such job |
| `413` | `too_large` | `input` is over 64 KB |
| `429` | `limit_reached` | The Build already has 20 unfinished jobs |
| `403` | `service_off` / `not_allowed` | Jobs are turned off on the platform, or the workspace has not allowed this Build to run them |

## Drive files in an installed copy

When another workspace installs your Stack, its copy of your Build can use the Files
service (`files.list`, `files.get`, `files.upload_url` and the rest) only if your Build
uses Drive folders beyond its own: you granted it a folder, or all of Drive, before you
packaged the Stack. The copy then reaches **the installing workspace's** folders, never
yours, and nothing it could reach in your workspace comes with it.

Files comes with the Stack: it is on when the Stack is installed. The copy's own folder
is **the Stack's folder** in the installing workspace, made at install, which it can
always read and write; leave `folder_id` out to use it. Every Build in your Stack shares
that folder and the same set of other folders. A member can add more of their folders on
the Stack's page, read or read and write, or switch Files off.

To give installers more than the Stack's folder, declare the folders your Build needs:
tell the Builder, for example *"installers get an Exports folder it can write"*. Each
declared folder is a name and a level (read unless you say write). It is part of your
Stack's submission, so Conducktor's review sees it, and at install it is found or made
at the top of the installing workspace's Workspace folder and granted to the copy. For a folder nobody added,
or while Files is off, Files answers `403 not_allowed` or finds nothing, so handle both
in your app.

## Network

On the default path Conducktor **tries** to apply a network policy to the Build. It is
best-effort: if applying it fails, the Build still launches, without the restrictions
below. Do not treat the policy as a guarantee. When it is applied:

- **Outbound:** DNS; the Conducktor platform; and **TCP 443 and TCP 22** to any public
  address (private address ranges are excluded). Other outbound ports are blocked, so a
  Build that talks to an external service on another port (a database on 5432, plain HTTP
  on 80) cannot reach it. An allowlist of declared connectors is planned and **not
  enforced yet**.
- **Inbound:** from the Conducktor platform, which routes traffic to the Build, on its
  serving port.

On the older bundle path the policy allows outbound **TCP 443 only**, also blocks
link-local addresses, and a failure to apply it fails the launch.

## Being called

A Build is a **plain HTTP service**; it does not speak MCP. When it is healthy:

1. Conducktor fetches its OpenAPI document from `GET /openapi.json` (FastAPI serves it for
   you) and registers each operation as a tool through **API-Bridge**. This step is
   best-effort: if the fetch or the registration fails, the launch still succeeds and no
   tools are registered until the Build is launched again.
2. The tools join the **Stack's connector**, the one connector that carries the tools of
   every Build in the Stack.
3. Assistants in the workspace, and MCP clients through `ask_conducktor`, can then call
   them. A tool call reaches your service as an ordinary HTTP request to the matching
   operation.

Your process serves HTTP on its serving port (the runtime's fixed port, or your `EXPOSE`
for the `dockerfile` runtime), with a valid `/openapi.json` and a health route (`/health`
for FastAPI). You never handle Conducktor's own authentication.

In practice:

- **Write operation summaries and descriptions for an AI reader.** Assistants choose tools
  by them, so say *when* to call each operation.
- **Keep `/openapi.json` accurate.** It decides which tools exist: an operation missing
  from the spec cannot be called, and a write operation with no request body schema is
  skipped at registration.
- **Expect concurrent calls.** Several Assistants can call the Build at once.

## What a Build cannot do yet

<!-- platform-contract:capabilities-roadmap -->

| Not yet available to app code | Workaround today | Status |
|------------------------------|------------------|--------|
| Read/write the workspace knowledge plane (Context Kernel) | Keep app state in the App Data Store | Roadmap |
| Read workspace Vault secrets | Copy the value you need into the app's own config with `PUT /api/builder/builds/{build_id}/config?workspace_id=<id>` | Roadmap |
| Drive a headless browser (the platform's Browser Automation tools) | Grant the Browser Automation connector to the Build and call its tools (navigate, screenshot, extract) as a connector call. To act signed in as the user, sign in first with the saved-login sign-in (`logins`, `services.sign_in`) and pass the browser session's context id it returns to those calls | Roadmap |

<!-- /platform-contract:capabilities-roadmap -->

When these become available they will use the same `CONDUCKTOR_APP_TOKEN`, so code
written against the token now will not need changes.

A Build runs as an HTTP service that stops when idle. For work on a schedule declare it in
`conducktor.tools.json`, and for long work use [Long-running jobs](#long-running-jobs).

## Resources

- Resources come from the runtime:

  <!-- platform-contract:runtime-resources -->

  | Runtime | CPU request / limit | Memory request / limit |
  |---------|---------------------|------------------------|
  | `dockerfile` | `200m` / `1000m` | `256Mi` / `1Gi` |
  | `nextjs` | `200m` / `1000m` | `512Mi` / `1Gi` |
  | `vite_static` | `100m` / `500m` | `256Mi` / `512Mi` |
  | `streamlit` | `250m` / `1000m` | `512Mi` / `1Gi` |
  | `fastapi` | `200m` / `1000m` | `256Mi` / `512Mi` |
  | `gradio` | `250m` / `1000m` | `512Mi` / `1Gi` |
  | `static_html` | `100m` / `500m` | `128Mi` / `512Mi` |

  <!-- /platform-contract:runtime-resources -->

  Both launch paths apply this table. There is no per-Build override, and no field to
  request one. A launch with `gpu=true` uses GPU resources instead, but only where GPU
  Builds are enabled, which they are not by default.
- A Build that needs more memory than its runtime's limit is stopped by the
  out-of-memory killer. Automatic memory growth after such a stop exists but is off by
  default; do not rely on it.
- The filesystem is temporary: anything you need after a restart goes in the data store.
- Base image: Conducktor picks a public base image for the runtime. A `Dockerfile` at the
  root of your package (or one directory below) selects the `dockerfile` runtime instead,
  and your `FROM` is used. See the [Package specification](APP_PACKAGE_SPEC.md).
