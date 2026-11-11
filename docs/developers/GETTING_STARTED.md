# Getting Started

> **Time:** about 15 minutes.
> **You need:** a Conducktor account and its user-session bearer token (see
> [Platform APIs § Authentication](PLATFORM_APIS.md#authentication)). Uploading needs a
> workspace administrator; as the owner of your workspace, you are one.

This walkthrough uploads a small HTTP service, launches it as a Build, and ends with its
operations registered as tools that your Assistants, and MCP clients through
`ask_conducktor`, can call.

> **How tools are found.** A launched Build is a plain HTTP service. After launch, Conducktor
> fetches its OpenAPI document from `/openapi.json` and registers each operation as a tool
> on the Build's Stack connector. So a Build needs to serve `/openapi.json` and a health
> route. FastAPI does both for you. A Build does not have to speak MCP.

## 1. Write the service

Two files. Neither is Conducktor-specific: there is no manifest to add.

The FastAPI runtime starts your code with `uvicorn main:app` on port 8000, so `main.py` must
define an object named `app` (other runtimes are in the
[Package specification](APP_PACKAGE_SPEC.md#runtimes)).

`main.py`:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/health")
def health():
    return {"status": "ok"}

@app.get("/hello", summary="Say hello to someone by name")
def hello(name: str = "world"):
    return {"message": f"hello, {name}"}
```

`requirements.txt`:

```
fastapi
uvicorn
```

Check it locally: run `uvicorn main:app`, then confirm `GET /health` returns 200 and
`GET /openapi.json` returns your spec.

## 2. Package it

```bash
zip -r hello-conducktor.zip main.py requirements.txt
```

No manifest, no Dockerfile, no secrets. The runtime is chosen from the files at the root of
the package (`requirements.txt` listing `fastapi` selects the FastAPI runtime). Shipping a
`Dockerfile` selects a different runtime; see the
[Package specification](APP_PACKAGE_SPEC.md#the-dockerfile-runtime).

## 3. Upload it

```bash
curl -X POST https://api.conducktor.com/api/builder/builds/upload \
  -H "Authorization: Bearer $CONDUCKTOR_TOKEN" \
  -F "file=@hello-conducktor.zip"
```

Conducktor validates the archive, puts your source in a Drive folder for the Build, and
returns the Build. The upload also gives the Build its own private Stack. When a runtime is
detected, the Build is ready to launch:

```json
{
  "build_id": "…",
  "conversation_id": "…",
  "drive_folder_id": "…",
  "app_name": "Hello Conducktor",
  "app_version": "1.0.0",
  "app_type": "mcp_server",
  "runtime_type": "fastapi",
  "file_count": 2,
  "skipped_files": [],
  "status": "succeeded",
  "launch_path": "/api/builder/builds/…/launch?workspace_id=…"
}
```

- `app_name` is the Build's name, made from the archive's file name
  (`hello-conducktor.zip` becomes "Hello Conducktor"). `app_version` is assigned by
  Conducktor and counts up for each upload of the same name in your workspace: the second
  upload is `1.1.0`. You cannot set either.
- `app_type` is always `mcp_server`. The value is historical; it does not change how the
  Build runs.
- When no runtime can be detected, the package is still stored (`201`), with
  `"status": "awaiting_build_recipe"`, `"runtime_type": null` and `"launch_path": null`.
  Conducktor then tries to work out how to run it, and the Build becomes `succeeded`
  (launchable) or `failed` with a reason. Launch refuses it until then.

Keep `build_id`. A rejected package gets an error that says why:

| Status | Meaning |
|--------|---------|
| `400` | Not a valid ZIP, or an unsafe one (path traversal, symlinks, ZIP64) |
| `413` | Over the [size caps](APP_PACKAGE_SPEC.md#package-layout) |
| `422` | A rejected compiled executable, a runtime marker file (such as `package.json`) that is not text, a file the malware scan flagged, or a detected runtime whose required files are missing |
| `429` | Over your workspace's build rate limit, which uploads share with Builder; retry later |

To add the Build to a Stack you already have instead of a new one, pass that Stack's id:
`POST /api/builder/builds/upload?stack_template_id=<id>`. The Stack must be one your
workspace built and has installed; otherwise the upload answers `404`.

## 4. Launch it

```bash
curl -X POST "https://api.conducktor.com/api/builder/builds/$BUILD_ID/launch" \
  -H "Authorization: Bearer $CONDUCKTOR_TOKEN"
```

Launch builds the container image and starts it in your workspace's isolated environment.
The response describes the running instance:

```json
{
  "build_id": "…",
  "instance_id": "…",
  "external_endpoint": "…",
  "runtime_type": "fastapi",
  "status": "running"
}
```

## 5. Call it

After launch, Conducktor reads your `/openapi.json` and registers each operation as a tool
on the Stack's connector. `GET /hello` is now a tool that:

- Assistants in your workspace can call;
- MCP clients connected to your workspace reach through `ask_conducktor`, at
  `https://mcp.conducktor.com/mcp`.

Assistants choose tools by their descriptions, so give each operation a clear `summary`.

Tool registration does not block the launch. If the service runs but `/openapi.json` is
missing or invalid, the Build still serves, and no tools are registered until you fix the
spec and launch again.

## Next

- [Runtime contract](APP_RUNTIME.md): environment, config and secrets, the data store, and
  calling the workspace's connectors.
- [Build and deploy](BUILD_AND_DEPLOY.md): every endpoint for following and operating a
  Build.
- [Stack authoring](STACK_AUTHORING.md): package your Build with Assistants and knowledge.
- [Flow triggers](PLATFORM_APIS.md#flow-triggers-inbound-events): start a Flow from your
  own systems.
- [API-Bridge](PLATFORM_APIS.md#api-bridge-openapi-to-tools): turn an existing REST API
  into tools without deploying code.
