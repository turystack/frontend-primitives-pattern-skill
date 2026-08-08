---
name: tury-stack-frontend-primitives-pattern
description: "UI primitive constitution — the unit grammar (HOW to write each primitive component) for turystack frontends. Read when building, modifying, or reviewing any UI primitive: in the shared component library or app-local (src/ui/). Covers component structure (5 files in the shared lib, 4 app-local — stories are lib-only), prop API design (semantic props, never className), tv() variant styling, headless composition, stories, tests, and consumption rules. Start at 00-overview (mental model + where the primitive lives + invariants), then read the exact section for what you touch. Each section splits into 🌐 Generic pattern (Concept / Pattern / Invariants table with stable XXX-n ids — the portable law) and 🛠️ Project-specific (Mechanisms / ✅ How to do it / ❌ Never do — the TypeScript·React·tailwind-variants code); reviews bind by id, the Invariants table + ❌ blocks are the gateable law."
---

# tury-stack-frontend-primitives-pattern — the constitution (HOW to write a primitive)

This skill is the **unit grammar** for turystack UI primitives: how each primitive component is written — in the shared library or app-local.

## How to use

1. **Detect where the primitive lives first.** Generic and reusable across products = the shared component library (`@turystack/react-web`). Product-specific = app-local `src/ui/`. **The grammar is identical in both** — same prop rules, same styling law; the single asymmetry is `.stories.tsx`, which only the shared library has (see `05-stories.md`). `00-overview.md` has the full table. Before creating anything, confirm the primitive (or the missing prop) doesn't already exist — extending the library always beats duplicating locally.
2. **Always read `00-overview.md` next** — the mental model (contract → styles → component → stories/tests → barrel) + the invariants most often broken. Keep it in context.
3. **Then read the exact section** for what you touch — never work from memory of these rules; they rot fast. Each section has two halves: **🌐 Generic pattern** (`Concept → Pattern → Invariants table`, each rule with a stable id `XXX-n`) and **🛠️ Project-specific** (`Mechanisms → ✅ How to do it → ❌ Never do`, in TypeScript/React/tailwind-variants). Read both — the 🌐 half is the portable law, the 🛠️ half is how it's written in this stack.
4. **The law is the `Invariants` table + the `❌ Never do` block, bound by id `XXX-n`.** Reviews bind to the **id**, not to a file/line — they read the invariant from 🌐 and the detector/exemplar from 🛠️. If a rule has a ❌ item, treat it as a hard reprove. `XXX-n` = **constitutional** (portable invariant); `XXX-Ln` = **stack lint** (a TS/React ergonomic that inverts in another stack — still enforced here).

> Comments in the examples are **didactic** — never copy them into code (zero-comment rule, same as the backend pattern).

## Routing — read the section for what you touch

| Touching | Read |
|---|---|
| Component folder / files / barrel | `01-component-structure.md` |
| Prop API (names, unions, modes, generics, onChange) | `02-props.md` |
| Styling / variants / theming (tv, tokens, theme in the provider) | `03-styles.md` |
| Composition (headless primitives, compound, asChild, provider) | `04-composition.md` |
| Storybook stories | `05-stories.md` |
| Component tests | `06-tests.md` |
| Consuming/extending primitives from an app | `07-consumption.md` |

`README.md` is the full index. For a new component, read every section before producing the first file.
