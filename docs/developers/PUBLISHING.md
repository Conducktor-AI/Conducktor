# Publishing a Stack

Any Stack you add to your workspace is yours to use there, with no review. To make one
available to every Conducktor workspace, send it to Conducktor for review from the
developer portal. If it passes, Conducktor adds it to the Stack library as a **third-party
Stack**.

Anyone with a Conducktor account can send a Stack for review. There is no application
step.

## What you can send

You can send a Stack you uploaded as a [Stack package](STACK_PACKAGE.md): a ZIP with
`stack.yaml` at its root (see [Stack authoring](STACK_AUTHORING.md)). Its Builds travel
with it through the manifest's `apps` section.

A Stack that Builder made for a project stays in its workspace and cannot be sent. To
publish code you built with Builder, list its Build in a manifest of your own.

## Before you send it

- [ ] The Stack uploads and installs cleanly into a workspace, and does what its
      `description` says.
- [ ] `name`, `tagline` and `description` tell a stranger what the Stack is for.
- [ ] Each specialist's `instructions` describe one clear job.
- [ ] `connectors.required` lists every connector the Stack needs, and nothing more.
- [ ] Each Build declares the connectors and Drive folders its installed copies use, and
      the most each may do (see
      [Runtime contract § Connectors in an installed copy](APP_RUNTIME.md#connectors-in-an-installed-copy)
      and [Drive files in an installed copy](APP_RUNTIME.md#drive-files-in-an-installed-copy)).
- [ ] Each setting an installer must fill has a `purpose`, and each secret setting a
      `reason` (see [Settings an installer fills](APP_RUNTIME.md#settings-an-installer-fills)).
- [ ] A Build that calls models handles `403 not_allowed`: AI is off in an installed copy
      until the installing workspace turns it on (see [Who pays](APP_RUNTIME.md#who-pays)).
- [ ] Nothing in the manifest or the package holds a secret, a token or personal data.

## Send it for review

1. Sign in to the developer portal at <https://app.conducktor.com/developers/stacks>. It
   lists the Stacks you added to your workspace.
2. Choose **Send for review** on the Stack.

Each Stack shows where it stands:

| Status | What it means | What you can do |
|--------|---------------|-----------------|
| **Draft** | In your workspace only | Send it for review |
| **In review** | Waiting for Conducktor's review | Wait. Sending again changes nothing |
| **Changes requested** | The review found something to change; the portal shows the note | Fix it, upload the package again with the same `slug`, then **Send for review again** |
| **In the Stack library** | Every workspace can install it | — |

## What the review looks at

Conducktor reviews what the Stack does, what it asks permission for (its connectors,
folders and settings, and why), and how it uses models.

## In the Stack library

Once the Stack passes, Conducktor's team adds it to the Stack library. It stays a third-party
Stack: your workspace remains its owner, and it never receives the permissions of
Conducktor's own Stacks. Every workspace can then install it and use it on the web, in
Conducktor's mobile apps, and from AI clients such as Claude and ChatGPT through
Conducktor.

While a Stack is in review or in the Stack library, uploading its package again is refused.
To change a published Stack, contact **developers@conducktor.com**.

If you delete your Conducktor account, your published Stacks are transferred to
Conducktor and retired: they leave the Stack library, the workspaces that installed them are
given 30 days' notice, and the Stacks are then removed.

## Pricing and payouts

How published Stacks are priced and how developers are paid is described at
<https://www.conducktor.com/developers>.

## Help

- Building and publishing Stacks: **developers@conducktor.com**
- If the portal says developer access is off for your account, contact the same address.
