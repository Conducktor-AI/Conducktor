# Product Designer

Turn a plain-language idea into a part brief a manufacturer can quote from.

> **Design only.** This example writes the brief. It does not generate CAD geometry or place orders. Those need a Build and commerce support, which come later.

## What it installs

- **Product Designer**, a manager-level Assistant that writes part briefs: purpose, key dimensions, load case, material, process, finish and open questions.
- **Design rules**: two starter knowledge documents, sheet-metal rules and a materials quick reference.
- **Draft a part brief**, a skill installed as a Flow.
- **An onboarding step** that prompts the user to describe their first part.

## Try it

Upload this folder's `stack.yaml` in a ZIP and install it (see the [Quickstart](../README.md#quickstart)). Then ask the Product Designer:

> A wall bracket for a 12 kg shelf, powder-coated, fixed with two screws.

## Make it yours

- Replace the starter rules with your own design-for-manufacturing knowledge. That is where most of this Stack's value lives.
- Tighten `instructions` to match the processes your manufacturers support.
- Change `slug` and `name` before you upload it.
