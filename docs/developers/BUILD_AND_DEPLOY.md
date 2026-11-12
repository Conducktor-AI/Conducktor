# Build and Deploy

> **Base URL:** `https://api.conducktor.com`. Every call sends
> `Authorization: Bearer <user-session token>`.

A Build goes from source to running service in two calls: **upload**, then **launch**.
[Getting started](GETTING_STARTED.md) walks through both with a worked example; this page
lists the endpoints.

## Get the source in

| Endpoint | What it takes |
|----------|---------------|
| `POST /api/builder/builds/upload` | A ZIP package as multipart `file` |
| `POST /api/builder/builds/from-drive/{document_id}?workspace_id=<id>` | A ZIP document already in your Drive |
| `POST /api/builder/builds/from-drive-folder/{folder_id}?workspace_id=<id>` | A top-level folder of Drive's Builds collection |
| `POST /api/builder/builds/from-huggingface?space_id=<owner/name>&workspace_id=<id>` | A Hugging Face Space |

All four need a workspace administrator (you, as your workspace's owner) and answer
`201`. The upload, Drive document and Hugging Face calls validate the package against the
[Package specification](APP_PACKAGE_SPEC.md) and return the same response: the Build's
`build_id`, its `status` (`succeeded` is launchable, `awaiting_build_recipe` is not yet)
and, when it is launchable, a `launch_path`. The Drive folder call copies nothing: the
folder you edit becomes the Build's source, and it returns `build_id`, `conversation_id`
and `runtime_type`.

Each new Build gets its own private Stack. To add it to a Stack you already built instead,
pass `stack_template_id=<id>` on the upload or Hugging Face call; a Stack that cannot take
another Build answers `404`.

## Launch

```
POST /api/builder/builds/{build_id}/launch
```

Launch builds the container image, starts it, and registers the Build's OpenAPI operations
as tools on its Stack's connector. It answers with the running instance
(`instance_id`, `external_endpoint`, `runtime_type`, `status: "running"`).

| Status | Meaning |
|--------|---------|
| `400` | The Build is not `succeeded` yet |
| `402` | Your workspace has reached a spending limit |
| `404` | No such Build in this workspace |
| `413` | The Build's source is too large; the response names the file |
| `422` | The source cannot be launched (for example, no runtime can be detected, or the Dockerfile's `EXPOSE` or build secrets are not valid) |
| `502` | The image build or the start failed; the response says why |

## Configure

Config and secrets for a running Build, delivered as environment variables at launch:

```
GET    /api/builder/builds/{build_id}/config?workspace_id=<id>
PUT    /api/builder/builds/{build_id}/config?workspace_id=<id>
DELETE /api/builder/builds/{build_id}/config/{key}?workspace_id=<id>
```

`PUT` replaces the whole list. See
[Runtime contract § Your own config and secrets](APP_RUNTIME.md#your-own-config-and-secrets).

To keep a Build running while it is idle, turn on always-on. The setting takes effect at
the next launch, and the Build's compute is billed while it is idle:

```
POST /api/builder/builds/{build_id}/always-on?workspace_id=<id>
{"always_on": true}
```

## Follow

```
GET  /api/builder/builds/active      # one row per Build project: its newest Build and status
GET  /api/builds/{build_id}/plan     # the plan Builder is following (Builder-made Builds)
GET  /api/builds/{build_id}/qa       # the QA report (Builder-made Builds)
POST /api/builder/builds/{build_id}/cancel
```

`GET /api/builder/builds/active` includes finished Builds by default; pass
`include_recent=false` for only those still in progress.

A Builder-made Build whose QA report passed can be made the live version with
`POST /api/builder/builds/{build_id}/promote`; without a passing report it answers `409`.

## Operate the running instance

```
POST /api/hosted/instances/{instance_id}/action
{"action": "stop"}
```

`action` is `start` (scale back up), `stop` (scale to zero) or `restart` (rolling restart).
A Build that is not always-on is scaled down when idle; `start` wakes it.

## Retired endpoints

The older `/api/apps` endpoints have been removed and answer `404`. Use the endpoints above:

| If you called | Use |
|---------------|-----|
| `POST /api/apps` | `POST /api/builder/builds/upload`; the Build is the record |
| `POST /api/apps/{id}/versions` | Upload again; each upload is the next Build |
| `POST /api/apps/{id}/deploy` | `POST /api/builder/builds/{build_id}/launch` |
| `/api/apps/deployments/*` | `POST /api/hosted/instances/{instance_id}/action` |
