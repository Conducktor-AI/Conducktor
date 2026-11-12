# Conducktor Stack examples

Ready-to-install example **Stacks** for [Conducktor](https://www.conducktor.com). Copy one, change it, and add it to your workspace in a minute.

A Stack is everything for one job, installed together: the Assistants that do the work, the knowledge they use, the Flows they run, the connectors they need and, optionally, Builds with their own code. You describe it in one file, `stack.yaml`.

| Example | What it adds | Good for learning |
| --- | --- | --- |
| [`hello-stack`](hello-stack/) | One Assistant and one knowledge document | The smallest valid Stack |
| [`product-designer`](product-designer/) | A Product Designer Assistant, design rules, a skill and an onboarding step | Specialists with expertise, linked knowledge, skills |

## Quickstart

1. Download one example's folder, for example [`hello-stack`](hello-stack/), and put its
   `stack.yaml` at the root of a ZIP file.
2. In Conducktor, open **Stacks**, choose **New Stack**, then **Import**, pick **ZIP file**,
   and choose your ZIP. The Stack is added to your workspace.
3. Find it in **Stacks** and choose **Install**.

A Stack you add is private to your workspace. To change it, edit `stack.yaml` and upload it
again with the same `slug`. When it is ready for everyone, see
[Publishing a Stack](../docs/developers/PUBLISHING.md).

## Contributing

Examples are welcome. Read [CONTRIBUTING.md](../CONTRIBUTING.md) first. Every example must be a valid `stack.yaml` that uploads and installs cleanly into a fresh workspace.

## Licence

The examples in this folder are [MIT licensed](LICENSE). Copy them freely into your own Stacks.
