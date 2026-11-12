# Package Specification

> **Applies to:** every way source reaches a Build: a ZIP upload
> (`POST /api/builder/builds/upload`, see [Getting started](GETTING_STARTED.md)), a ZIP
> document or folder in Drive, a Hugging Face Space, and code Builder writes with you.

A package is your source code as it already is. **There is no manifest to add.**
Conducktor works out how to build and run it from the files at the root. One optional
file, `conducktor.tools.json`, declares scheduled jobs and webhooks; see
[Runtime contract § Scheduled jobs and webhooks](APP_RUNTIME.md#scheduled-jobs-and-webhooks).

> **A `stack.yaml` at the root makes the ZIP a Stack.** Conducktor then reads it as a
> [Stack package](STACK_PACKAGE.md), and your Builds go in its `builds/` folder.

> **`gt.yaml` is not read.** Older packages carried a `gt.yaml` at the root. It is no
> longer required or read: a package that still has one uploads normally, the file is kept
> as ordinary source, and nothing in it has any effect.

## Package layout

```
my-service.zip
├── main.py            # your source (FastAPI: defines an `app` object)
├── requirements.txt   # Python dependencies (or package.json for Node)
└── ...                # any other source files and assets
```

Rules:

- An archive that wraps everything in one top-level folder is accepted; the folder is
  removed. So `git archive` output and a GitHub "Download ZIP" work as they are.
- Your source must follow the fixed layout of its [runtime](#runtimes). Every runtime
  except `dockerfile` starts your code with a fixed command on a fixed port; you do not
  name an entrypoint.
- 🔴 **A `Dockerfile` at the root, or one directory below it, selects the `dockerfile`
  runtime**, which builds your own Dockerfile instead of a managed image. Moving it to
  `docker/Dockerfile` does not avoid this; two levels down (`services/api/Dockerfile`)
  does. If you want a managed runtime, do not ship a Dockerfile there. See
  [The `dockerfile` runtime](#the-dockerfile-runtime).
- Do not put secrets in the package. Set config and secrets with
  `PUT /api/builder/builds/{build_id}/config?workspace_id=<id>`; they are delivered to the
  running Build as environment variables. The launch request itself takes no environment
  variables. See [Runtime contract § Your own config and secrets](APP_RUNTIME.md#your-own-config-and-secrets).

Upload limits, checked at upload:

- At most <!-- platform-contract:package-upload-limits -->**100 MB** compressed, **~487 MB** uncompressed, **5000 files** maximum<!-- /platform-contract:package-upload-limits -->. (The
  uncompressed figure leaves room for the framing the build archive adds. A Build's Drive
  folder can keep growing after upload, as you and Builder add files, and the launch
  checks the source size again.)
- **ZIP64 archives are rejected** with `400`. Re-zip without ZIP64 (`zip -r` and
  `git archive` both stay in the classic format well inside these limits).
- **Text and binary files are both kept**: images, fonts and vendored archives are packaged
  as they are. Empty files are kept. File permission bits are kept, so an executable
  `entrypoint.sh` stays executable.
- **Some compiled executables are rejected** with `422` naming the file, for example Java
  archives (`.jar`). The check goes by file name, so it guards against packaging a binary
  by accident; it is not a security boundary. Shell scripts are always kept. Prefer
  shipping source and letting the build produce binaries: a binary built for another
  platform is the most common reason a container builds and then does not start.
- Symlinks, absolute paths and `..` path segments are rejected.
- `conducktor_app.py` at the root is reserved for the platform SDK (see
  [Runtime contract § The SDK](APP_RUNTIME.md#the-sdk)); a copy in your package is skipped
  and listed in the upload response's `skipped_files`.
- Uploads count against your workspace's **build rate limit**, which it shares with
  Builder. Over the limit, the upload answers `429`; retry later.

**How the runtime is chosen.** The first match wins:

1. a `Dockerfile` at the root or one directory below: `dockerfile`;
2. `package.json`: `nextjs` when it depends on `next`, otherwise `vite_static`;
3. `requirements.txt`: `streamlit`, `fastapi` or `gradio`, by which one it lists, and
   `fastapi` when it lists none;
4. `index.html`: `static_html`.

A package that matches none is still accepted, with `status: "awaiting_build_recipe"`.
Conducktor then tries to work out how to build it; until the Build is `succeeded`, launch
refuses it with `400`.

## Runtimes

Every runtime except `dockerfile` has a **fixed** start command and port:

<!-- platform-contract:runtime-ports -->

| Runtime | Detected from | Port | Health path | Your source must provide |
|---------|---------------|------|-------------|--------------------------|
| `dockerfile` | a `Dockerfile` at the package root or one directory below it (checked first) | the first `EXPOSE` of the final stage | `/` | your own `Dockerfile` |
| `nextjs` | `package.json` that depends on `next` | 3000 | `/` | a `"build"` script in `package.json` |
| `vite_static` | any other `package.json` | 8080 | `/` | a `"build"` script in `package.json` |
| `streamlit` | `requirements.txt` that lists `streamlit` | 8501 | `/_stcore/health` | `app.py` at the root |
| `fastapi` | `requirements.txt` that lists `fastapi`, or lists none of the Python frameworks | 8000 | `/health` | `fastapi` and `uvicorn` in `requirements.txt`; `main.py` exporting an `app` object; a `/health` route; only when the screen is a built project (`screen/package.json`, for example Vite with React): a `"build"` script that writes it to `screen/dist/`. Launch builds it and serves `screen/dist/` as `static/` |
| `gradio` | `requirements.txt` that lists `gradio` (and not `streamlit` or `fastapi`) | 7860 | `/` | `app.py` at the root |
| `static_html` | `index.html` (checked last) | 8080 | `/` | `index.html` at the root |

<!-- /platform-contract:runtime-ports -->

The `dockerfile` runtime takes its port from your `EXPOSE` (see
[The `dockerfile` runtime](#the-dockerfile-runtime)).
The `fastapi` runtime starts your code with <!-- platform-contract:fastapi-start-command -->`uvicorn main:app --host 0.0.0.0 --port 8000`<!-- /platform-contract:fastapi-start-command -->, and FastAPI
serves `/openapi.json` for you.

**Tools come from `/openapi.json`, whatever the runtime.** After a launch, Conducktor
requests `GET /openapi.json` from the Build and, if it gets a valid spec, registers its
operations as tools on the Stack's connector. A missing or invalid spec is skipped and the
launch still succeeds. UI runtimes (`streamlit`, `gradio`, `nextjs`, `vite_static`,
`static_html`) usually serve a surface and no spec, so usually register no tools. For a
service whose operations should be tools, use the FastAPI shape below.

The required layout is checked at upload, and a package missing it is rejected with `422`:

- FastAPI: `main.py`, and `fastapi` and `uvicorn` in `requirements.txt`.
- Streamlit: `app.py` at the root (started with `streamlit run app.py`).
- Gradio: `app.py` at the root (started with `python app.py`).
- Node (`nextjs`, `vite_static`): a `"build"` script in `package.json`. With
  `package-lock.json` (or `npm-shrinkwrap.json`) the build runs `npm ci`; without one it
  runs `npm install`, and the upload response warns that versions may drift between
  builds. Packages that use **yarn or pnpm** are rejected with a message naming the fix
  (`npm install --package-lock-only`); the build installs with npm.

Minimal FastAPI shape:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/health")
def health():
    return {"status": "ok"}

@app.get("/lookup_contact", summary="Find a contact by email address")
def lookup_contact(email: str):
    ...
```

Every uploaded package is recorded with the type `mcp_server` (the `app_type` field of the
upload response). The name is historical: the Build is a plain HTTP service and does not
speak MCP. There is no other type to choose. A Build that must run a job on a schedule
declares it instead (see
[Scheduled jobs and webhooks](APP_RUNTIME.md#scheduled-jobs-and-webhooks)).

## The `dockerfile` runtime

A package with its own `Dockerfile` (see [Package layout](#package-layout)) is built from
that file. In short:

<!-- platform-contract:dockerfile-rules -->

- Any `FROM` is allowed; no base image is enforced.
- The serving port is the first `EXPOSE` of the final stage: a literal number from 1024 to 65535; a `${VARIABLE}` cannot be read.
- Readiness is an HTTP `GET /` on the serving port.
- On the default launch path the container runs as UID 1000 / GID 1000 with all Linux capabilities dropped and a writable root filesystem.
- Build secrets reach the build only through `RUN --mount=type=secret,id=<KEY>`; every mounted id (unless marked `required=false`) must be configured as a non-empty `build` entry.

<!-- /platform-contract:dockerfile-rules -->

In more detail:

- **Your image does not run as its own `USER`.** On the default launch path every
  runtime, this one included, runs as UID 1000 / GID 1000 (`fsGroup` 1000) with
  `runAsNonRoot`, no privilege escalation, and all Linux capabilities dropped; the root
  filesystem is writable. So your entrypoint never runs as root: an entrypoint that
  `chown`s as root and then drops to another user (the `su-exec` / `gosu` pattern) cannot
  do either step. Anything your image writes at run time must be writable by UID 1000.
- **The port comes from `EXPOSE`.** The first `EXPOSE` of your final stage is the port
  your service listens on. It must be a literal number from <!-- platform-contract:dockerfile-expose-range -->**1024 to 65535**<!-- /platform-contract:dockerfile-expose-range -->: a
  `${VARIABLE}` cannot be read, and a port below <!-- platform-contract:dockerfile-expose-min -->1024<!-- /platform-contract:dockerfile-expose-min --> cannot be bound, because the
  container runs as a non-root user with no `NET_BIND_SERVICE` capability. An
  out-of-range `EXPOSE` is refused at launch with `422`.
- **A platform shim fronts your service.** When your final stage declares an `ENTRYPOINT`
  or `CMD`, launch adds a stage that runs a platform shim in front of it. The shim serves
  on port 8199 (8299 if your service uses 8199).
  - With an `EXPOSE`, the shim passes every request to your service and answers `503`
    until your service is listening. The one exception is `/condk/state`, which the shim
    always answers itself, so that path is **reserved**. The shim also sets `PORT` to your
    `EXPOSE` port for your process.
  - Without a readable `EXPOSE` (none, or a `${VARIABLE}`), your image is treated as a
    command-line tool: the shim answers requests itself, including `/`, `/openapi.json`,
    and a `/run` operation that runs your command.
  - When the shim is not added (no `ENTRYPOINT` or `CMD` in the final stage, or a
    Dockerfile the shim cannot read), your service runs directly, and a readable,
    in-range `EXPOSE` is **required**; without one, launch is refused with `422`.
- **Readiness is `GET /`.** On the default launch path the same check is the startup
  probe (up to about 5 minutes: every 10 seconds, 30 failures), then the readiness and
  liveness probes; a liveness failure restarts the container. Behind the shim that request
  reaches **your** `/`, so a service whose `/` does not answer successfully (an API with
  no root route, for example) never becomes ready. A TCP check is not available yet.

### Build secrets

To use a credential during the build (a private package index token, say) without
putting it in the image, mount it in a `RUN` step with
`RUN --mount=type=secret,id=<KEY> …`, and store `<KEY>` with
`PUT /api/builder/builds/{build_id}/config?workspace_id=<id>` as a secret entry of kind
`build`. A build secret reaches the build only, never the running Build.

Before the build starts, Conducktor reads your Dockerfile and **refuses the launch with
`422`** if a secret mount in the final stage, or in a stage it builds from, names an id
that is not configured, is configured but empty, or cannot be stored under that name. An
id must be a valid environment variable name: letters, digits and underscores, not
starting with a digit (rename `npm-token` to `NPM_TOKEN`, for example). A mount marked
`required=false` is not checked. A build secret you configure but never mount is not an
error.

## Build limits

- A build that runs longer than <!-- platform-contract:build-timeout -->**30 minutes**<!-- /platform-contract:build-timeout --> is stopped and fails.

## Runtime environment

- A Build runs as a container in an isolated, workspace-scoped environment.
- The base image depends on the runtime. Each managed runtime builds `FROM` a public
  image: <!-- platform-contract:runtime-base-images -->`nextjs` `node:22-bookworm-slim` (built on `node:22-bookworm-slim`); `vite_static` `nginxinc/nginx-unprivileged:alpine` (built on `node:22-bookworm-slim`); `streamlit` `python:<version>-slim`; `fastapi` `python:<version>-slim`; `gradio` `python:<version>-slim`; `static_html` `nginxinc/nginx-unprivileged:alpine`<!-- /platform-contract:runtime-base-images -->.
  `<version>` is the Python version your repository declares, if it is supported, and
  otherwise the platform default. The `dockerfile` runtime uses whatever `FROM` you write.
  The launch build passes no build arguments.
- Resources are set per runtime (<!-- platform-contract:runtime-resources-example -->for example `fastapi` gets `256Mi` memory requested and `512Mi` as its limit; the `dockerfile` runtime `256Mi` / `1Gi`<!-- /platform-contract:runtime-resources-example -->); see
  [Runtime contract § Resources](APP_RUNTIME.md#resources). There is no way to request
  different ones.
- Treat the filesystem as temporary. For durable state, use the
  [data store](APP_RUNTIME.md#data-store--your-durable-state).
- What the running Build gets (environment, token, SDK) is in the
  [Runtime contract](APP_RUNTIME.md).

## Checklist

Before you upload:

- [ ] No manifest to add. The name comes from the archive's file name, the version is
      assigned for you, and the runtime is chosen from the root files.
- [ ] Scheduled jobs and webhooks, if any, are declared in `conducktor.tools.json` at
      the root.
- [ ] The source follows its runtime's layout (FastAPI: `main.py` defining `app`).
- [ ] No secrets and no `.git` directory in the archive.
- [ ] **No `Dockerfile`** at the root or one level down, unless you mean to own the build.
- [ ] Locally, the service answers its health route with 200 and serves a valid
      `/openapi.json`.
