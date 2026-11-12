# Stack Package

You add a Stack to your workspace by uploading one ZIP file. A ZIP can be one of two things:

| The ZIP holds | Conducktor creates |
|---------------|--------------------|
| Source code, with no `stack.yaml` at its root | A **Build** in its own private Stack (see [Package specification](APP_PACKAGE_SPEC.md)) |
| A `stack.yaml` at its root | A **Stack**, with everything the package describes |

This page specifies the second kind: a **Stack package**. One package carries a whole
Stack: working Assistants with their instructions, expertise, Skills and Flows, the
knowledge they use, the Builds that do the work, and the connectors they need.

## Upload it

In Conducktor, open **Stacks**, choose **New Stack**, then **Import**, pick **ZIP file**,
and choose your package. Conducktor checks `stack.yaml` and the package straight away and
refuses a package that breaks a rule, naming the field, file or folder at fault.

A package with no Builds creates the Stack at once. A package with Builds shows
**Building** while Conducktor builds and launches each one in the background; the Stack
appears once every Build has launched. Either way it is **private to your workspace** and
in **Draft**: you can install it, use it, and later send it for review (see
[Publishing a Stack](PUBLISHING.md)).

The upload is all or nothing. If any Build fails to build or launch, or has not launched
within 60 minutes, no Stack is created, the Builds made for that upload are deleted, and
the upload names the Build at fault.

A package is uploaded from **New Stack** only. A ZIP with `stack.yaml` at its root
uploaded through **Add a Build** on an existing Stack is refused.

## Layout

```
freelance-designer.zip
├── stack.yaml                           required: the manifest
├── specialists/
│   └── brand-strategist/                one folder per Assistant, named by its slug
│       ├── instructions.md              who it is and how it works
│       ├── expertise.md                 optional: deeper method, always in context
│       └── skills/                      optional: procedures it loads when needed
│           └── brand-audit/
│               └── SKILL.md
├── flows/
│   └── weekly-brand-report/             optional: one folder per Flow, named by its key
│       └── instructions.md
├── knowledge/
│   └── brand/voice.md                   one file per knowledge document
└── builds/
    └── invoice-api/                     one folder per Build, each a package of its own
        ├── main.py
        └── requirements.txt
```

A ZIP that wraps everything in one top-level folder is accepted, as for a Build, so
`git archive` output and a GitHub "Download ZIP" work as they are. Only `stack.yaml` is
required. Other files at the top level, such as a README or a LICENSE, are allowed and are
not part of the Stack. Every folder is optional: a Stack that only hires existing Conducktor Assistants
needs nothing but the manifest.

`stack.yaml` declares every part of the Stack. The folders hold the long text and the
code. The manifest's sections are specified in [Stack authoring](STACK_AUTHORING.md); this
page specifies the files and how they fit together.

## A complete Assistant

An Assistant you define is called a **specialist**. On install it becomes an Assistant in
the user's workspace, hired and ready to chat, to run its Flows, and to call tools. A
complete specialist is made of these parts:

| Part | What it does | Where it goes |
|------|--------------|---------------|
| Identity | Name, role and description the user sees | `specialists[]` in `stack.yaml` |
| Instructions | Who the Assistant is and how it works. Required. | `specialists/<slug>/instructions.md`, or `instructions` inline |
| Expertise | Deeper method and judgement, in context on every turn | `specialists/<slug>/expertise.md`, or `expertise` inline |
| Skills | Step-by-step procedures it loads only when a request needs one | `specialists/<slug>/skills/<skill>/SKILL.md` |
| Flows | Repeatable work the user runs on demand or on a schedule | `custom_flows` in `stack.yaml`, each naming its `specialist`; long instructions in `flows/<key>/instructions.md` |
| Knowledge | Reference documents it can search | `knowledge/<path>`, linked by the specialist's `knowledge` list |
| Tools | What it can act with: your Builds' operations and the user's connectors | `apps` and `connectors.required` in `stack.yaml` |

### Identity

```yaml
specialists:
  - slug: brand-strategist                # [a-z0-9-], 3–50, unique in the Stack
    name: Brand Strategist
    role: Brand strategy & positioning    # a short label
    description: |                        # optional, up to 1,000 characters; shown to the user
      Finds your positioning, writes your messaging, and audits your brand.
    level: manager                        # intern | manager | executive (default intern)
    knowledge: [brand/voice.md]           # optional; paths from `knowledge`
```

`level` sets how capable and how autonomous the Assistant is in the installing workspace,
and how it is billed.

### Instructions and expertise

`instructions.md` tells the Assistant who it is, what it is for, and how it should work:
its focus, its tone, what it should always and never do. Write it to the Assistant, in
the second person.

`expertise.md` holds the method behind the work: the frameworks it applies, how it
judges quality, the mistakes it avoids. Expertise is added to the instructions under an
`## Expertise` heading, so both are in context on every turn. Instructions and expertise
together are capped at 64 KB per specialist. Keep them to what the Assistant needs on every
turn, and put longer procedures in Skills.

### Skills

A Skill is one procedure: the steps to follow for one kind of task, such as "audit a
brand" or "write a positioning statement". The Assistant does not carry its Skills in
every turn. When a request fits one, the Skill is offered to the Assistant, and it loads
the procedure and follows it. An Assistant can have many Skills without slowing down or
costing more on turns that need none.

Each Skill is a folder under `specialists/<slug>/skills/`, named by the Skill's key
(`[a-z0-9-]`, 3–64 characters), holding one `SKILL.md` that starts with `---` front
matter:

```markdown
---
name: Brand audit
description: Audit a brand's positioning, voice and visual identity against its goals,
  and return a scored report with the three changes that matter most.
---

1. Ask for the brand's website, its goals for the year, and its two closest competitors,
   unless the conversation already has them.
2. Read the brand voice guide in knowledge.
3. ...
```

- `name`: up to 200 characters.
- `description`: what the Skill does and when to use it, up to 1,000 characters. This is
  what the request is matched against, so say plainly which requests it fits.
- The body, after the front matter, is the procedure: Markdown, up to 64 KB.
- Any other front matter key is refused, so a misspelled one fails the upload instead of
  being ignored.

A new or changed Skill becomes available to match requests a few minutes after install,
once Conducktor has indexed it. An update to the Stack refreshes its Skills in place and
retires the ones the new version drops.

Up to 100 Skills per specialist. A Skill belongs to its specialist only; another
Assistant cannot load it.

### Flows

A Flow is repeatable work the user can see, run on demand, or put on a schedule, such as
"Weekly brand report". Declare your own Flows in `custom_flows`, and give each one the
`specialist` that owns it:

```yaml
custom_flows:
  weekly-brand-report:
    name: Weekly brand report
    description: Summarise this week's brand mentions and what to do about them
    specialist: brand-strategist
    # instructions: in flows/weekly-brand-report/instructions.md
    inputs:                              # optional; a JSON Schema object
      type: object
      properties:
        week: { type: string, description: "The week to report on, e.g. 2026-W46" }
    tools: [list_mentions]               # optional; tools the Flow needs
    outputs:
      description: A one-page report
      saves_document: true               # the result is saved to Drive
```

| Field | Required | Meaning |
|-------|----------|---------|
| `name` | ✅ | The Flow's name, up to 255 characters |
| `slug` | — | Up to 128 characters |
| `description` | — | What the Flow does |
| `specialist` | — | The `slug` of one of the package's specialists. The Flow installs on that Assistant. Without it, the Flow installs on Conducktor's own Flows tab. |
| `instructions` | — | What the Assistant does when the Flow runs, inline or in `flows/<key>/instructions.md` (the folder holds only that file). Defaults to the description. |
| `inputs` | — | A JSON Schema object for what the user fills in when they run it |
| `tools` | — | Tool names the Flow needs, including your Builds' operations. A tool the workspace does not have makes the Flow unavailable on install, with the reason. |
| `outputs.description` | — | What it produces |
| `outputs.saves_document` | — | `true` to save the result to Drive |
| `tags` | — | Labels |

Up to 100 Flows per Stack, each at most 64 KB. (`skills` is the older name for
`custom_flows` and is still accepted; a manifest with both is refused.) A Build can also declare scheduled jobs and
webhooks that become Flows (see
[Runtime contract § Scheduled jobs and webhooks](APP_RUNTIME.md#scheduled-jobs-and-webhooks)).

### Tools

Each Build in the package serves HTTP operations, and when it launches in the installing
workspace, each operation becomes a tool on the Stack's connector, which the workspace's
Assistants can call. To give your Assistant something to act with, write a Build, and
describe each operation well in its OpenAPI document: that description is what the
Assistant reads when it decides which tool to call.

List the outside services your Assistants need in `connectors.required`; the installer
connects their own accounts.

## Knowledge

List each document in `stack.yaml` with its `title` and `path`, and put its text in
`knowledge/<path>`:

```yaml
knowledge:
  - title: Brand Voice Guide
    path: brand/voice.md
```

Files are UTF-8 text (Markdown recommended): 256 KB per document and 100 documents per
Stack. A specialist's `knowledge` list links it to the documents it should use.

## Builds

Each folder under `builds/` is a complete Build package, laid out exactly as the
[Package specification](APP_PACKAGE_SPEC.md) describes for a package's root, and listed in
`apps`:

```yaml
apps:
  - path: builds/invoice-api
```

Each folder needs a runtime Conducktor can detect from its files (see
[Package specification § Runtimes](APP_PACKAGE_SPEC.md#runtimes)); a folder with none is
refused, naming it. Conducktor builds and launches each one in your workspace before it
creates the Stack, and the Stack pins the image it built. A `path` entry works only inside
a package. An `apps` entry can instead name a Build you already have by `build_id` (see
[Builds in a Stack](STACK_AUTHORING.md#builds-in-a-stack)).

## Rules

- **Declared once.** Text set both inline in `stack.yaml` and in a file is refused, naming
  both.
- **Nothing orphaned.** A folder under `specialists/` that no specialist names, a folder
  under `flows/` that `custom_flows` does not declare, a file
  under `knowledge/` that no entry names, and a folder under `builds/` that `apps` does
  not list are each refused.
- **Paths.** Relative only: no absolute paths, no `..`, no symbolic links.
- **Size.** The whole package follows the Build upload limits in the
  [Package specification](APP_PACKAGE_SPEC.md): at most 100 MB compressed and 5000 files.
  Up to 20 Builds and 50 specialists per Stack.
- **Rate limit.** The whole package counts as one upload against your workspace's build
  rate limit, however many Builds it holds.
- **No secrets.** Never put a secret, a token or personal data anywhere in the package.
  Settings an installer must provide are declared by your Builds (see
  [Runtime contract § Settings an installer fills](APP_RUNTIME.md#settings-an-installer-fills)).

## Revise

Upload the package again with the **same `slug`** in `stack.yaml`. The Stack is updated in
place and reset to Draft, and installed copies show **Update available**. The Builds in
the package are built afresh, and the previous upload's Builds are removed once the
revised Stack is saved. A Stack that is in review or in the Stack library cannot be replaced
this way.

## What an installing workspace sees

Your Assistants' know-how is yours. The people who install your Stack use it; they do
not get to read it.

| Part | The installer |
|------|---------------|
| A specialist's name, role and description | Sees them |
| Its instructions and expertise | Never sees them, in the app or through the API |
| Its Skills | Sees which Skill was used; never sees the procedure |
| A Flow | Sees its name, description and inputs, and can run and schedule it |
| Knowledge | Can read it: documents are placed in their Drive |
| A Build | Uses its screen and its tools; it runs from the image built at upload |

Put nothing in knowledge that you do not want the installer to read.

## Before you upload

- [ ] `stack.yaml` is at the root (or inside one top-level folder) and every section is
      valid (see [Stack authoring](STACK_AUTHORING.md)).
- [ ] Each specialist has instructions, and its expertise holds only what it needs on
      every turn.
- [ ] Each Skill's `description` says plainly which requests it fits.
- [ ] Each Flow names its `specialist`, and every tool it lists is served by one of your
      Builds or by a connector in `connectors.required`.
- [ ] Each Build folder launches on its own as a Build (see
      [Getting started](GETTING_STARTED.md)).
- [ ] Nothing in the package holds a secret, a token or personal data.
