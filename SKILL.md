---
name: turystack-frontend-primitives-pattern
description: "The unit grammar for Turystack UI primitives — how ONE primitive component is written, in the shared library (@turystack/react-web, react-mobile) or app-local (src/ui/). Covers the component's file set, prop API design (semantic props, discriminated modes, controlled/uncontrolled pairs, onChange delivering the value), tv() variant styling with zero public className, headless and compound composition, theming through the provider, stories, tests, and how an app consumes or extends a primitive. Use it whenever you build, modify or review a primitive — a Button, Input, Select, Modal, Table or any shared UI unit — whenever an existing one needs a new variant, size, state or prop, and especially when a request sounds like 'style this differently', 'pass a className', 'wrap this component' or 'make a custom version', which are the four ways a primitive's contract gets broken. For screens, routes, forms and data surfaces use turystack-frontend-pattern; for the law behind both, turystack-architecture-pattern."
---

# turystack-frontend-primitives-pattern

The **unit grammar** for Turystack UI primitives: how each primitive component
is written, in the shared library or app-local.

## Where this sits

```text
turystack-architecture-pattern     the portable law (ARC-… ids)
└── turystack-frontend-pattern     the app: features, routes, forms, surfaces
    └── this skill                  one primitive: its contract, styles, tests
```

A primitive is the unit a feature composes with. This skill governs the unit;
the frontend skill governs what is built out of them. When a rule here and a
constitutional law disagree, the constitution wins.

## How to use

1. **Decide where the primitive lives first.** Generic and reusable across
   products → the shared component library (`@turystack/react-web`).
   Product-specific → app-local `src/ui/`. **The grammar is identical in both**
   — same prop rules, same styling law; the single asymmetry is
   `.stories.tsx`, which only the shared library has (`05-stories.md`).
   `00-overview.md` has the full table.
   Before creating anything, confirm the primitive — or the missing prop —
   does not already exist. Extending the library always beats duplicating
   locally, because a duplicate is a second contract nobody will keep in sync.
2. **Read `00-overview.md` next** — the mental model (contract → styles →
   component → stories/tests → barrel) plus the invariants most often broken.
   Keep it in context.
3. **Then read the exact section for what you touch.** Never work from memory of
   these rules; they are dense and they rot. For a brand-new component, read
   every section before producing the first file.

## How a section is written

Each section splits in two, and the split is the point:

- **🌐 Generic pattern** — `Concept → Pattern → Invariants`, each rule with a
  stable id. The **Invariants** table is the portable law and is what a review
  binds to.
- **🛠️ Project-specific** — `Mechanisms → ✅ How to do it → ❌ Never do`, in
  TypeScript · React 19 · headless primitives · tailwind-variants · Tailwind
  CSS v4. A stack swap rewrites only this half.

`XXX-n` is **constitutional** (it would survive a stack swap); `XXX-Ln` is a
**stack lint** (it exists because of this toolchain and would invert elsewhere —
still enforced here). An item in a **❌ Never do** block is a hard reprove, and
it carries the id it violates. Comments in the examples are didactic — never
copy them into code; the standard is zero comments, same as the backend skill.

Every section opens with a **Rules defined here** line naming the ids it owns,
so you can confirm you opened the right file before reading it. A section
that says `none` states no law of its own — everything in it is cited.

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
| Accessible name, state exposure, focus, the reason channel of a blocked control | `08-accessibility.md` |

`README.md` is the full index.

## Before you finish

1. **Contract first.** Is every value domain a named exported union, and does
   `onChange` deliver the value rather than the DOM event? (`PROP-*`)
2. **No escape hatch.** Are `className` and `style` still absent from the public
   props, with the whole visual resolve internal? (`STY-*`)
3. **File set.** Does the component have the complete set for its context —
   `.tsx`, `.types.ts`, `.test.tsx`, `index.ts`, plus `.stories.tsx` in the
   library — and does the barrel export component and types? (`CMP-*`)
4. **Accessible base.** Is every interactive element a headless accessible
   primitive rather than raw HTML, with a name channel, its states exposed and
   a visible focus ring? (`CPS-1`, `AXS-1`, `AXS-2`, `AXS-3`)
5. **State behavior.** Does each state boolean have its compound effect tested —
   `loading` shows an indicator *and* blocks interaction, `disabled` blocks
   `onClick`? (`TST-2`)
6. **Duplication.** Did this add a primitive the library already has, or a prop
   that should have been added upstream instead? (`USO-*`)

Each id above carries a gate binding in its Invariants table, and
`turystack-proof` prints this same list with real pass/fail — `manual` bindings
stop for a person to sign. Binding kinds: `turystack-architecture-pattern` ›
`00-overview.md` › *Gate*.

## Ownership rule

This skill answers **how a primitive is written**. The frontend skill answers
**what the app builds with it**. The constitution answers **why the boundary
exists**. When they disagree, that order runs backwards: the constitution wins,
then the frontend skill, then this one.
