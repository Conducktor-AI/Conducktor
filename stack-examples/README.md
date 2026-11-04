# Conducktor Stack examples

Ready-to-install example **Stacks** for [Conducktor](https://www.conducktor.com). Copy one, change it, and add it to your workspace in a minute.

A Stack is everything for one job, installed together: the Assistants that do the work, the knowledge they use, the Flows they run, the connectors they need and, optionally, Builds with their own code. You describe it in one file, `stack.yaml`.

| Example | What it adds | Good for learning |
| --- | --- | --- |
| [`hello-stack`](hello-stack/) | One Assistant and one knowledge document | The smallest valid Stack |
| [`product-designer`](product-designer/) | A Product Designer Assistant, design rules, a skill and an onboarding step | Specialists with expertise, linked knowledge, skills |

## Quickstart

```bash
git clone https://github.com/Conducktor-AI/Conducktor.git
cd Conducktor/stack-examples/hello-stack

curl -X POST https://api.conducktor.com/api/stacks/manifest \
  -H "Authorization: Bearer $CONDUCKTOR_TOKEN" \
  -F "file=@stack.yaml"
```

The response includes a `template_id`. Install the Stack into your workspace:

```bash
curl -X POST https://api.conducktor.com/api/stacks/templates/$TEMPLATE_ID/install \
  -H "Authorization: Bearer $CONDUCKTOR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"workspaceId": "'"$WORKSPACE_ID"'"}'
```

A Stack you add is private to your workspace. To change it, edit `stack.yaml` and post it again with the same `slug`.

## Contributing

Examples are welcome. Read [CONTRIBUTING.md](../CONTRIBUTING.md) first. Every example must be a valid `stack.yaml` that installs cleanly into a fresh workspace.

## Licence

The examples in this folder are [MIT licensed](LICENSE). Copy them freely into your own Stacks.
