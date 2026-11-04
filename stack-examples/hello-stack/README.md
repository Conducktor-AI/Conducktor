# Hello Stack

The smallest useful Stack: one Assistant and one knowledge document.

## What it installs

- **Writing Coach**, a new Assistant that rewrites drafts to be clearer and shorter.
- **House Style**, a knowledge document the Writing Coach uses.

## Try it

From this folder:

```bash
curl -X POST https://api.conducktor.com/api/stacks/manifest \
  -H "Authorization: Bearer $CONDUCKTOR_TOKEN" \
  -F "file=@stack.yaml"
```

Then install it with the `template_id` from the response (see the [main README](../README.md#quickstart)), open the Writing Coach and paste in a draft.

## Make it yours

- Change `instructions` to give the Assistant a different job.
- Add more documents under `knowledge` and list their `path`s on the specialist.
- Change `slug` and `name` before you post it, so it does not replace this example in your workspace.
