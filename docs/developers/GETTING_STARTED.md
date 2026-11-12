# Getting Started

> **Time:** about 15 minutes.
> **You need:** a Conducktor account.

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

In Conducktor, open **Stacks**, choose **New Stack**, then **Import**, and pick **ZIP file**.
Choose `hello-conducktor.zip`.

Conducktor checks the archive, keeps your source in a Drive folder for the Build, and
gives the Build its own private Stack. When it can tell which runtime to use, it builds
the container image and launches the Build for you.

- The Build is named after the archive (`hello-conducktor.zip` becomes "Hello
  Conducktor"). Its version is assigned for you and goes up with each upload of the same
  name: the second upload is `1.1.0`.
- When no runtime can be detected, the package is still saved, and Conducktor works out how
  to run it. The Build can be launched once that succeeds.
- A package that cannot be accepted is refused with the reason: an unsafe or invalid ZIP,
  a package over the [size limits](APP_PACKAGE_SPEC.md#package-layout), a rejected compiled
  executable, or a runtime's required files missing. Uploads also share your workspace's
  build rate limit with Builder; if you reach it, try again later.

To add the Build to a Stack you already have, open that Stack and add the Build from there
instead of using **New Stack**.

## 4. Launch it

The upload launches the Build when it can. Launching builds the container image and starts
it in your workspace's isolated environment. You can launch it again from its Stack in
**Stacks** at any time, for example after you upload a new version.

## 5. Call it

After launch, Conducktor reads your `/openapi.json` and registers each operation as a tool
on the Stack's connector. `GET /hello` is now a tool that:

- Assistants in your workspace can call;
- AI clients connected to your workspace, such as Claude and ChatGPT, reach through
  `ask_conducktor` at `https://mcp.conducktor.com/mcp`.

Assistants choose tools by their descriptions, so give each operation a clear `summary`.

Tool registration does not block the launch. If the service runs but `/openapi.json` is
missing or invalid, the Build still serves, and no tools are registered until you fix the
spec and launch again.

## Next

- [Runtime contract](APP_RUNTIME.md): environment, config and secrets, the data store, and
  calling the workspace's connectors.
- [Build and deploy](BUILD_AND_DEPLOY.md): every endpoint for following and operating a
  Build.
- [Scheduled jobs and webhooks](APP_RUNTIME.md#scheduled-jobs-and-webhooks): run your
  Build on a schedule, or when an outside service sends an event.
- [Stack authoring](STACK_AUTHORING.md) and [Stack package](STACK_PACKAGE.md): ship your
  Build with Assistants and knowledge as one Stack.
- [Publishing a Stack](PUBLISHING.md): send your Stack for review so every workspace can
  install it.
- [API-Bridge](PLATFORM_APIS.md#api-bridge-openapi-to-tools): turn an existing REST API
  into tools without deploying code.
