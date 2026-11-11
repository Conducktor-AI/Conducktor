# Stack Authoring

> **Base URL:** `https://api.conducktor.com`

A **Stack** is everything for one job, installed together: the **Builds** that do the work
(see [Package specification](APP_PACKAGE_SPEC.md)), the **Assistants** that run it,
**Flows**, the **connectors** it needs, and **knowledge** documents.

There are two ways to make one:

- **Builder.** Describe what you want in Conducktor. Builder makes a private Stack for the
  project and proposes its parts; launching the project is its install.
- **A manifest**, described on this page. You write one file, `stack.yaml`, and post it.
  Conducktor creates a Stack **private to your workspace**, which you can then install.

## The manifest (`stack.yaml`)

```yaml
schema_version: 1
slug: freelance-designer                 # unique in your workspace; lowercase [a-z0-9-], 3–50
name: Freelance Designer
icon: "🎨"
tagline: Run your design business on autopilot
description: |
  Everything a solo designer needs to run the business side.

connectors:                              # existing catalog connectors
  required: [figma, notion, gmail]

roster:                                  # EXISTING catalog Assistants, hired on install
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

skills:                                  # custom skills, installed as Flows
  invoice-client:
    name: Invoice a client
    description: Draft and send an invoice from a brief
    workflow: { }                        # optional workflow definition

apps:                                    # YOUR workspace's Builds, pinned by id
  - build_id: "8f0c2d4e-…"               # a succeeded, launched Build of yours

knowledge:                               # put in the workspace's Drive on install
  - title: Brand Voice Guide
    path: brand/voice.md
    content: |
      Our tone is warm, concise, and confident...
  - title: Pricing Playbook
    path: pricing/playbook.md
    content: |
      Anchor high, offer three tiers...

flows:                                   # catalog Flows, installed as Flows of their Assistants
  - client-onboarding

onboarding:                              # optional guided-setup steps
  - title: Connect your tools
    description: Link Figma and your email so your team can act.
    required_connectors: [figma, gmail]  # a step with connectors gates the install;
                                         # with none, the install is immediate

pricing:
  price_cents: 0                         # must be 0; paid Stacks are not supported yet
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
| `slug` | ✅ | `[a-z0-9-]`, 3–50 characters, unique within your workspace (another workspace may use the same slug). It cannot match a catalog Stack's slug, or a Stack Builder made in your workspace. The slug is a label; the Stack's identity is its `template_id`. |
| `name` | ✅ | Display name, up to 100 characters |
| `icon` | — | An emoji (default 📦) |
| `tagline` / `description` | — | Shown on the Stack |
| `connectors.required` | — | **Existing** catalog connector slugs, shown as "connect these" on install. An unknown slug is rejected with `422`, and so is a connector that belongs to a Platform Stack (it arrives only by installing that Stack). |
| `roster` | — | **Existing** catalog Assistant slugs, hired into the workspace. An unknown slug is rejected with `422`. |
| `roster_required` | — | Slugs from `roster` the installer may not deselect. Each must also be in `roster`. |
| `specialists` | — | **New** Assistants you define, created in the installing workspace (see [Specialists](#specialists)) |
| `skills` | — | Custom skills, installed as **Flows** on Conducktor's Flows tab, one per skill, named by the skill's `name`. Up to 100 skills; each definition at most 64 KB serialized. Skill ids at most 100 characters; each needs a `name` (at most 255 characters); optional `slug` at most 128 characters. A skill whose Flow name is already taken on Conducktor's tab by a Flow that is not this Stack's, whose tools are not in the workspace, or whose inputs a Flow cannot take is reported *unavailable* (`custom_skills_unavailable` / `skillsUnavailable`), not overwritten. A reinstall refreshes this Stack's Flows in place unless you edited one. |
| `apps` | — | **Your own** Builds, pinned by id (see [Builds in a Stack](#builds-in-a-stack)). Each entry is `{build_id: <id>}`. Up to 20; a Build listed twice is rejected. |
| `knowledge` | — | Text documents put in the installing workspace's Drive (see [Knowledge](#knowledge)) |
| `pipeline` | — | How documents in your Stack's corpus are processed (see [Corpus processing pipeline](#corpus-processing-pipeline)). Needs `roster_required` or `specialists`. |
| `flows` | — | Catalog Flow slugs. Each must be an **existing, active** catalog Flow; an unknown slug is rejected with `422`. |
| `onboarding` | — | Guided-setup steps. A step's `required_connectors` must be existing catalog connector slugs (unknown ones are rejected with `422`). The setup runs only when at least one step lists connectors; the install happens at the **last** such step, once every listed connector is connected. Steps without connectors are informational. |
| `pricing.price_cents` | — | Must be `0`; a nonzero price is rejected |

### Builds in a Stack

`apps` lists Builds from **your own workspace** by their `build_id`. When you post the
manifest, each Build must:

- belong to your workspace (a Build from another workspace is rejected);
- have `status` `succeeded` and a built image;
- have been **launched at least once**, and its launched instance must serve HTTP and have
  a recorded runtime.

Otherwise the manifest is rejected with `422` saying which condition failed. The Stack pins
exactly that Build's image: later changes to the Build do not reach installs until you post
the manifest again.

On install, each Build is **started** in the installing workspace from the pinned image,
and its OpenAPI operations are registered as tools on the Stack's connector. A Build that
cannot be started is reported in `apps_unavailable`, with a reason, and the rest of the
install continues.

### Knowledge

Each `knowledge` entry becomes a text document in the installing workspace's **Stack
Knowledge** collection in Drive, in a folder named for your Stack. The specialists and
roster Assistants your Stack installs can find these documents when they work.

- `title`: the document's name.
- `path`: a relative path such as `brand/voice.md`; folders are created as needed. No
  absolute paths and no `..`.
- `content`: UTF-8 text (Markdown recommended), up to 256 KB per document and 100
  documents per Stack.
- `department`: optional; controls which specialists are linked to it. Defaults to
  `general`.

### Specialists

`roster` hires **existing** catalog Assistants; `specialists` defines **new** ones. Each
specialist becomes an Assistant in the installing workspace, hired and ready.

- `slug`: unique within your Stack (`[a-z0-9-]`, 3–50 characters).
- `name`: display name.
- `role`: a short label for what the specialist does.
- `level`: `intern`, `manager` or `executive` (default `intern`).
- `instructions`: what the specialist is told (how it thinks, its focus, its approach).
  This becomes the Assistant's system prompt. (`persona` is the older name for this field
  and is still accepted; if you send both and they differ, the manifest is refused.)
- `expertise`: optional; deeper method, appended to the instructions under an
  `## Expertise` heading. Instructions and expertise together are capped at 64 KB; up to
  50 specialists per Stack.
- `knowledge`: optional list of knowledge `path`s from your `knowledge` section. A path
  that is not one of your slices is rejected. **Scope:** today this links the specialist to
  the workspace's whole Stack Knowledge collection, which holds the documents of *every*
  installed Stack, so it can find more than the paths you listed.

The specialists you write are your own content and live in the installing workspace.

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
check `specialists_unavailable` in the install result. Uninstalling removes the pipeline;
it cannot be changed through collection settings. Declaring `claims` turns on fact
extraction (model calls) for documents processed in that corpus after the pipeline is
written. Files already in the corpus are not reprocessed when the pipeline is added or
changed. An existing install does not pick up a changed pipeline by itself: post the
manifest again, then re-run the install's steps (see [Revise](#revise)).

## Add the Stack to your workspace

Upload the file:

```bash
curl -X POST https://api.conducktor.com/api/stacks/manifest \
  -H "Authorization: Bearer $CONDUCKTOR_TOKEN" \
  -F "file=@stack.yaml"
```

or send it as JSON:

```bash
curl -X POST https://api.conducktor.com/api/stacks/manifest \
  -H "Authorization: Bearer $CONDUCKTOR_TOKEN" \
  -H "Content-Type: application/json" \
  -d @stack.json
```

The optional `workspaceId` query parameter names the owning workspace; it defaults to
yours, and must be yours. A manifest larger than 32 MB is refused with `413`.

Response (`201`):

```json
{
  "slug": "freelance-designer",
  "name": "Freelance Designer",
  "template_id": "…",
  "owner_workspace_id": "…",
  "review_status": "draft",
  "visibility": "workspace",
  "components": {
    "roster": 2, "specialists": 1, "connectors": 3, "surfaces": 0,
    "skills": 1, "knowledge_slices": 2, "flows": 1
  },
  "component_status": {
    "roster": "installed_on_install",
    "specialists": "installed_on_install",
    "connectors": "surfaced_on_install",
    "surfaces": "none",
    "skills": "installed_on_install",
    "knowledge": "installed_on_install",
    "flows": "installed_on_install"
  },
  "install_path": "/api/stacks/templates/…/install",
  "submit_path": "/api/stacks/templates/…/submit"
}
```

`template_id` is the Stack's id; keep it. `component_status` says what each component
does on install. `submit_path` is deprecated; see [Who can see your Stack](#who-can-see-your-stack).

Errors: `400` (no manifest, not UTF-8, unparseable, or no workspace), `403` (not your
workspace), `409` (the slug is taken by a catalog Stack or by a Stack in your workspace
that cannot be replaced), `413` (too large), `422` (a field is invalid; the response names
the `field`).

## Install it

```bash
curl -X POST https://api.conducktor.com/api/stacks/templates/$TEMPLATE_ID/install \
  -H "Authorization: Bearer $CONDUCKTOR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"workspaceId": "'"$WORKSPACE_ID"'"}'
```

Optionally send `rosterSlugs` to hire only some of the roster (members of
`roster_required` cannot be left out).

| Component | On install |
|-----------|------------|
| Roster (existing Assistants) | **Hired** into the workspace |
| Specialists (new Assistants) | **Created** as workspace Assistants, hired and ready |
| Custom skills | **Installed** as Flows on Conducktor's Flows tab |
| Knowledge | **Written** to the workspace's Stack Knowledge collection in Drive |
| Corpus processing pipeline | **Written** to your Stack's corpus collections when an owning Assistant was installed |
| Builds (`apps`) | **Started** from the pinned image, with their tools on the Stack's connector |
| Connectors | **Shown** as "connect these"; the user connects their own accounts |
| Flows | **Installed** on the Flows tab of the Assistant each one belongs to (Conducktor's, when the catalog Flow names none). A Flow whose Assistant was left out of the install, or whose Assistant already has a Flow of that name, is skipped and reported in `flows_skipped`. |

**When the install happens.** A Stack goes through guided setup only when there is
something to connect: `connectors.required` with no custom `onboarding` steps (setup then
adds a connect step for you), or custom steps of which at least one lists
`required_connectors`. The install then happens at the **last** step that lists
connectors, once all of them are connected. Any other Stack, including one whose custom
steps are only informational, **installs at once**. Custom steps replace the generated
connect step, so connectors in `connectors.required` alongside informational-only steps
are not shown; put them in a step's `required_connectors`.

An immediate install answers with the installation's `id`, `status`, `skillsUnavailable`
and a `cascade` object holding what each part did, including `flows_skipped`,
`apps_installed` and `apps_unavailable`.

**Uninstall** reverses the install: the roster is un-hired, the specialists you wrote are
removed, the knowledge documents are archived, the Flows the install made are removed, and
its Builds are stopped. Anything still used by another installed Stack stays, and so does
anything the user changed after the install (an edited Flow, a revised document): uninstall
never destroys the user's own work.

## Revise

Post the corrected manifest again with the **same `slug`**. The Stack is updated in place:
the same `template_id`, reset to `draft`. To apply the change to an existing install, re-run
its steps:

```bash
curl -X POST https://api.conducktor.com/api/stacks/installations/$INSTALLATION_ID/install-skills \
  -H "Authorization: Bearer $CONDUCKTOR_TOKEN"
```

## Who can see your Stack

A Stack you post is **private to your workspace**. `GET /api/stacks/templates` lists the
catalog plus your workspace's own Stacks; each entry's `owner_workspace_id` is your
workspace for yours and `null` for catalog Stacks.

Catalog Stacks (Platform Stacks) are added by Conducktor. A Stack your workspace makes
stays in your workspace; it is not promoted to the catalog. The `submit` endpoint is
deprecated: do not build on it.

## Lifecycle summary

```
write stack.yaml
  → POST /api/stacks/manifest                         create the Stack (private, draft)
  → POST /api/stacks/templates/{id}/install           install it: Assistants hired and created,
                                                      Flows and knowledge added, Builds started,
                                                      at once or after guided setup
  → POST /api/stacks/manifest (same slug)             revise it in place
  → POST /api/stacks/installations/{id}/install-skills  apply the revision to an install
  → uninstall                                         reversed, keeping the user's own edits
```
