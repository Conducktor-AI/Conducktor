# Stack Authoring

> **Base URL:** `https://api.conducktor.com`

A **Stack** is everything for one job, installed together: the **Builds** that do the work
(see [Package specification](APP_PACKAGE_SPEC.md)), the **Assistants** that run it,
**Flows**, the **connectors** it needs, and **knowledge** documents.

There are two ways to make one:

- **Builder.** Describe what you want in Conducktor. Builder makes a private Stack for the
  project and proposes its parts; launching the project is its install.
- **A manifest**, described on this page. You write one file, `stack.yaml`, and upload it
  in a ZIP, on its own or as a [Stack package](STACK_PACKAGE.md) with your Builds,
  specialists' instructions and knowledge in folders.
  Conducktor creates a Stack **private to your workspace**, which you can then install.
  A manifest Stack is also the kind you can send for review, so that every workspace can
  install it from the Stack library (see [Publishing a Stack](PUBLISHING.md)).

## The manifest (`stack.yaml`)

```yaml
schema_version: 1
slug: freelance-designer                 # unique in your workspace; lowercase [a-z0-9-], 3–50
name: Freelance Designer
icon: "🎨"
tagline: Run your design business on autopilot
description: |
  Everything a solo designer needs to run the business side.

connectors:                              # existing Conducktor connectors
  required: [figma, notion, gmail]

roster:                                  # EXISTING Conducktor Assistants, hired on install
  - brand-strategist
  - social-media-manager
roster_required: [brand-strategist]      # roster members the installer may not deselect

specialists:                             # NEW Assistants you define, created on install
  - slug: brand-strategist-pro
    name: Brand Strategist Pro
    role: Brand strategy & positioning
    level: manager                       # intern | manager | executive
    instructions: |                      # what the specialist is told
      You are a senior brand strategist. You think in positioning,
      category design, and jobs-to-be-done.
    expertise: |                         # optional; appended under a "## Expertise" heading
      Frameworks you apply: JTBD, category design, the StoryBrand arc.
    knowledge: [brand/voice.md]          # optional; must name a knowledge slice below
    connectors: [hubspot]                # optional; the connectors it starts with
    isolated: false                      # optional; true = works only from its own files, chats and memory

custom_flows:                            # your own Flows
  invoice-client:
    name: Invoice a client
    description: Draft and send an invoice from a brief
    specialist: brand-strategist-pro     # optional; the Flow installs on this specialist
    instructions: |                      # what the Assistant does when the Flow runs
      Draft the invoice from the brief, show it for approval, then send it.

apps:                                    # Builds, pinned at upload
  - path: builds/invoice-api             # a Build folder in the same ZIP (see Stack package)
  - build_id: "8f0c2d4e-…"               # or a succeeded, launched Build you already have

knowledge:                               # put in the workspace's Drive on install
  - title: Brand Voice Guide
    path: brand/voice.md
    content: |
      Our tone is warm, concise, and confident...
  - title: Pricing Playbook
    path: pricing/playbook.md
    content: |
      Anchor high, offer three tiers...

flows:                                   # Conducktor Flows, installed as Flows of their Assistants
  - client-onboarding

onboarding:                              # optional guided-setup steps
  - title: Connect your tools
    description: Link Figma and your email so your team can act.
    required_connectors: [figma, gmail]  # a step with connectors gates the install;
                                         # with none, the install is immediate

pricing:
  price_cents: 0                         # optional; must be 0 (no purchase price)
```

> **Refused fields.** These are rejected with `422` in a manifest you write: `surfaces`
> (navigation surfaces are reserved for Conducktor's own Stacks), `connectors.categories`
> (use `connectors.required`), and `category` / `tags` (not stored yet). **Unknown
> top-level keys are refused** with `422` naming the key, so a misspelled section fails
> instead of disappearing.

### Field reference

| Field | Required | On install |
|-------|----------|------------|
| `schema_version` | — | Must be `1` |
| `slug` | ✅ | `[a-z0-9-]`, 3–50 characters, unique within your workspace (another workspace may use the same slug). It cannot match the slug of a Stack in the Stack library, or a Stack Builder made in your workspace. The slug is a label; the Stack's identity is its `template_id`. |
| `name` | ✅ | Display name, up to 100 characters |
| `icon` | — | An emoji (default 📦) |
| `tagline` / `description` | — | Shown on the Stack |
| `connectors.required` | — | **Existing** Conducktor connector slugs, shown as "connect these" on install. An unknown slug is rejected with `422`, and so is a connector that belongs to one of Conducktor's own Stacks (it arrives only by installing that Stack). |
| `roster` | — | **Existing** Conducktor Assistant slugs, hired into the workspace. An unknown slug is rejected with `422`. |
| `roster_required` | — | Slugs from `roster` the installer may not deselect. Each must also be in `roster`. |
| `specialists` | — | **New** Assistants you define, created in the installing workspace (see [Specialists](#specialists)) |
| `custom_flows` | — | Your own **Flows**, one per entry, named by its `name`: installed on the specialist an entry names with `specialist: <slug>` (one of this Stack's `specialists`), otherwise on Conducktor's Flows tab. Fields are listed in [Stack package § Flows](STACK_PACKAGE.md#flows). Up to 100; each definition at most 64 KB serialized. Keys at most 100 characters; each needs a `name` (at most 255 characters); optional `slug` at most 128 characters. A Flow whose name is already taken on its tab by a Flow that is not this Stack's, whose tools are not in the workspace, or whose inputs a Flow cannot take is reported as unavailable on the install, not overwritten. A reinstall refreshes this Stack's Flows in place unless you edited one. `skills` is the older name for this section and is still accepted; a manifest with both is refused. |
| `apps` | — | Builds, pinned at upload (see [Builds in a Stack](#builds-in-a-stack)). Each entry is `{path: builds/<folder>}` for a Build in the same package (only inside a ZIP), or `{build_id: <id>}` for one you already have. Up to 20; a Build listed twice is rejected. |
| `knowledge` | — | Text documents put in the installing workspace's Drive (see [Knowledge](#knowledge)) |
| `pipeline` | — | How documents in your Stack's corpus are processed (see [Corpus processing pipeline](#corpus-processing-pipeline)). Needs `roster_required` or `specialists`. |
| `flows` | — | Conducktor Flow slugs. Each must be an **existing, active** Conducktor Flow; an unknown slug is rejected with `422`. |
| `onboarding` | — | Guided-setup steps. A step's `required_connectors` must be existing Conducktor connector slugs (unknown ones are rejected with `422`). The setup runs only when at least one step lists connectors; the install happens at the **last** such step, once every listed connector is connected. Steps without connectors are informational. |
| `pricing.price_cents` | — | Optional. Must be `0`: a Stack has no purchase price, and a nonzero value is rejected. How published Stacks earn is described at <https://www.conducktor.com/developers>. |

### Builds in a Stack

An `apps` entry with a `path` names a Build folder in the same package: Conducktor builds
and launches it at upload (see [Stack package § Builds](STACK_PACKAGE.md#builds)).

An entry with a `build_id` names a Build already in **your own workspace**. When you
upload the manifest, each such Build must:

- belong to your workspace (a Build from another workspace is rejected);
- have `status` `succeeded` and a built image;
- have been **launched at least once**, and its launched instance must serve HTTP and have
  a recorded runtime.

Otherwise the manifest is rejected with `422` saying which condition failed. The Stack pins
exactly that Build's image: later changes to the Build do not reach installs until you upload
the manifest again.

On install, each Build is **started** in the installing workspace from the pinned image,
and its OpenAPI operations are registered as tools on the Stack's connector. A Build that
cannot be started is reported on the install, with a reason, and the rest of the
install continues.

### Knowledge

Each `knowledge` entry becomes a text document in the installing workspace's **Stack
Knowledge** collection in Drive, in a folder named for your Stack. The specialists and
roster Assistants your Stack installs can find these documents when they work.

- `title`: the document's name.
- `path`: a relative path such as `brand/voice.md`; folders are created as needed. No
  absolute paths and no `..`.
- `content`: UTF-8 text (Markdown recommended), up to 256 KB per document and 100
  documents per Stack. In a [Stack package](STACK_PACKAGE.md) you can leave `content` out
  and put the text in `knowledge/<path>` instead.
- `department`: optional; controls which specialists are linked to it. Defaults to
  `general`.

### Specialists

`roster` hires **existing** Conducktor Assistants; `specialists` defines **new** ones. Each
specialist becomes an Assistant in the installing workspace, hired and ready.

- `slug`: unique within your Stack (`[a-z0-9-]`, 3–50 characters).
- `name`: display name.
- `role`: a short label for what the specialist does.
- `description`: optional, up to 1,000 characters; what it helps with, shown to the user.
  Without it, the role is shown.
- `level`: `intern`, `manager` or `executive` (default `intern`).
- `instructions`: what the specialist is told (how it thinks, its focus, its approach).
  This becomes the Assistant's system prompt. (`persona` is the older name for this field
  and is still accepted; if you send both and they differ, the manifest is refused.)
- `expertise`: optional; deeper method, appended to the instructions under an
  `## Expertise` heading. In a [Stack package](STACK_PACKAGE.md), instructions and
  expertise can live in `specialists/<slug>/instructions.md` and `expertise.md`, and the
  specialist can have Skills: procedures it loads when a request needs one (see
  [Stack package § A complete Assistant](STACK_PACKAGE.md#a-complete-assistant)). Instructions and expertise together are capped at 64 KB; up to
  50 specialists per Stack.
- `knowledge`: optional list of knowledge `path`s from your `knowledge` section. A path
  that is not one of your slices is rejected. **Scope:** today this links the specialist to
  the workspace's whole Stack Knowledge collection, which holds the documents of *every*
  installed Stack, so it can find more than the paths you listed.
- `isolated`: optional, `true` or `false` (default `false`). An isolated specialist works
  only from its own files, its own chats and its own memory. It does not use the
  workspace's shared memory, it reads the workspace's shared folder only if the person
  gives it access, and nothing it learns is shared with the workspace's other Assistants. Use it when your specialist handles material that should
  stay with it, such as one person's records. Things to know:
  - The installer is told, before they install, which of your specialists are isolated.
  - It is set when the specialist is first created in a workspace. Changing it in a later
    version of your Stack does not change a specialist that is already installed.
  - An isolated specialist cannot be added to another Stack's team, and it is left out
    of group conversations with other Assistants.
  - It must be written as a real boolean: `isolated: "true"` (a string) is refused.

- `connectors`: optional list of connector slugs this specialist works with. Without it,
  the specialist can use every connector in the installing workspace. With it, the
  specialist starts with only the workspace's connections of those connectors switched on;
  the rest, and connectors added later, start off. People can switch any of them on or off
  in the **Connectors** section of the specialist's page.

The specialists you write run in the installing workspace. Their instructions and
expertise are never shown to the people who install your Stack.

### Corpus processing pipeline

A Stack can declare how documents in its corpus are processed: which facts to extract,
which passages to extract from, and how extracted facts are combined.

```yaml
pipeline:
  version: 1                     # an integer of at least 1; increase it on every change
  claims: { ... }                # optional: the claim schema sent with every extraction request
  filtration: { ... }            # optional: which passages are worth extracting from
  consolidation:                 # optional
    conflict_key_fields: [vendor, invoice_number]
```

- Declare at least one of `claims`, `filtration` or `consolidation`. Any other key inside
  `pipeline` is refused, naming it.
- `claims` must be a non-empty mapping of at most 16,384 bytes as UTF-8 JSON (measured with
  a space after each `,` and `:`).
- `filtration` is checked for its top-level shape, and a malformed one is refused with the
  problem named. Nested entries are not checked yet, so write each `term_lists` entry as
  `{label, terms}`.
- `consolidation.conflict_key_fields` is a list of 1–20 field names with no surrounding
  whitespace.
- A pipeline needs an Assistant to own the corpus: declare `roster_required` or
  `specialists`, or the manifest is refused.

On install the pipeline is written to your Stack's corpus collections, provided an
Assistant that owns the corpus was installed (a `roster_required` member hired, or at
least one specialist created). If every specialist is skipped as unavailable and there is
no `roster_required`, the pipeline is not written and the install does not warn yet, so
check that the install created your specialists. Uninstalling removes the pipeline;
it cannot be changed through collection settings. Declaring `claims` turns on fact
extraction (model calls) for documents processed in that corpus after the pipeline is
written. Files already in the corpus are not reprocessed when the pipeline is added or
changed. An existing install does not pick up a changed pipeline by itself: upload the
manifest again, then apply the update to the install (see [Revise](#revise)).

## Add the Stack to your workspace

Put `stack.yaml` at the root of a ZIP file, with any Builds, specialist files and
knowledge files laid out as a [Stack package](STACK_PACKAGE.md). In Conducktor, open **Stacks**, choose
**New Stack**, then **Import**, pick **ZIP file**, and choose your ZIP. Conducktor reads
the manifest and creates the Stack, **private to your workspace** and in **Draft**.

A manifest that cannot be accepted is refused with the reason, naming the field where
there is one: a file that is not valid YAML or UTF-8, a `slug` already used by a Stack in the
Stack library or by a Stack in your workspace that cannot be replaced, a field that breaks a rule
above, or a package over the [size limits](STACK_PACKAGE.md#rules). A Build in the package
that fails to build refuses the whole upload.

## Install it

Open **Stacks**, find your Stack and choose **Install**.

| Component | On install |
|-----------|------------|
| Roster (existing Assistants) | **Hired** into the workspace |
| Specialists (new Assistants) | **Created** as workspace Assistants, hired and ready |
| Custom Flows | **Installed** on the specialist each names, or on Conducktor's Flows tab |
| Knowledge | **Written** to the workspace's Stack Knowledge collection in Drive |
| Corpus processing pipeline | **Written** to your Stack's corpus collections when an owning Assistant was installed |
| Builds (`apps`) | **Started** from the pinned image, with their tools on the Stack's connector |
| Connectors | **Shown** as "connect these"; the user connects their own accounts |
| Flows | **Installed** on the Flows tab of the Assistant each one belongs to (Conducktor's, when the Conducktor Flow names none). A Flow whose Assistant was left out of the install, or whose Assistant already has a Flow of that name, is skipped. |

**When the install happens.** A Stack goes through guided setup only when there is
something to connect: `connectors.required` with no custom `onboarding` steps (setup then
adds a connect step for you), or custom steps of which at least one lists
`required_connectors`. The install then happens at the **last** step that lists
connectors, once all of them are connected. Any other Stack, including one whose custom
steps are only informational, **installs at once**. Custom steps replace the generated
connect step, so connectors in `connectors.required` alongside informational-only steps
are not shown; put them in a step's `required_connectors`.

Whatever a component could not do is reported on the install, for example a Flow skipped
because its Assistant was left out, or a Build that could not be started.

**Uninstall** reverses the install: the roster is un-hired, the specialists you wrote are
removed, the knowledge documents are archived, the Flows the install made are removed, and
its Builds are stopped. Anything still used by another installed Stack stays, and so does
anything the user changed after the install (an edited Flow, a revised document): uninstall
never destroys the user's own work.

## Revise

Upload the corrected package with the **same `slug`**. The Stack is updated in
place: the same `template_id`, reset to Draft. A Stack that is in review or already in the
Stack library cannot be replaced this way. Your installed copy then shows **Update
available**; apply the update to bring it to the new version.

## Who can see your Stack

A Stack you upload is **private to your workspace**, and you can use it there with no
review. Nobody outside your workspace can see it.

To make a Stack available to every workspace, send it for review from the developer
portal. If it passes, Conducktor adds it to the Stack library as a **third-party** Stack: it
stays yours, and it carries your workspace as its owner. You never publish to the
Stack library directly. See [Publishing a Stack](PUBLISHING.md).

## Lifecycle summary

```
write stack.yaml (and any Builds, specialist files, knowledge files)
  → upload the ZIP (Stacks → New Stack → Import)       Builds built, the Stack created,
                                                       private, in Draft
  → Install                                            Assistants hired and created, Flows and
                                                       knowledge added, Builds started, at once
                                                       or after guided setup
  → upload again with the same slug                    revised in place
  → uninstall                                          reversed, keeping the user's own edits
  → developer portal: Send for review                  reviewed, then added to the Stack library
```
