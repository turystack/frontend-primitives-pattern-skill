# UI Primitive Standards

> **Purpose.** Generic pattern for writing **UI primitive components**: the grammar of how each primitive is built — prop contract, styling by variants, composition, living doc and test. **Each section splits into two parts:**
> - **🌐 Generic pattern** — the **portable law**: Concept + Pattern + an **Invariants** table indexing each rule by a stable id `XXX-n`. Holds in any UI stack (React, Vue, etc.); the text stays true after swapping frameworks.
> - **🛠️ Project-specific** — the **code** that implements each rule in the current stack: Mechanisms per rule + ✅ How to do it + ❌ Never do. Swapping stacks rewrites **only** this part.
>
> Reviews bind by **id** (`XXX-n`), never by file/line: they read the invariant in the 🌐 part and the detector/example in the 🛠️ part.
>
> **Markers:**
> - `XXX-n` **constitutional** — portable invariant (holds in any stack). `XXX-Ln` **stack lint** — exists only because of the current language/lib ergonomics and inverts in another stack; still id-registered and enforced.
>
> **Reference stack (🛠️ part):** TypeScript · React 19 · headless primitives (`@base-ui/react`) · tailwind-variants (`tv`) · Tailwind CSS v4 · Storybook · Vitest + Testing Library · Biome via `@turystack/frontend-config`. Style: 2 spaces · single quotes · no semicolons.
>
> **⚠️ Comments in the examples are didactic** — they explain the rule being demonstrated there. **Never copy a comment into the code**: the standard is zero comments.

**Rules defined here:** none — every rule this file states is defined
elsewhere and cited by id.

## Context (check BEFORE writing any component)

A primitive lives in one of two places — **the grammar is identical in both**:

| | Shared library (`@turystack/react-web`) | App-local (the app's `src/ui/`) |
|---|---|---|
| When | generic, reusable across products | product-specific |
| Structure | `src/components/<name>/` | `src/ui/<name>/` |
| Files | 5 — `.tsx` · `.types.ts` · `.stories.tsx` · `.test.tsx` · `index.ts` | 4 — the same ones, without `.stories.tsx` |
| Living doc | Storybook — one story per variation | `.types.ts` + real usage in the routes |
| Exports | the library's root barrel (`src/index.ts`) | the `src/ui/index.ts` barrel |
| Before creating | confirm it does not exist in the barrel | confirm it exists neither in the library **nor** in `src/ui/` |

**Order of preference:** new prop/variant on an existing primitive → new primitive in the library (if generic) → app-local primitive in `src/ui/` (if specific) → **duplicating, never**.

## Mental model

**Build flow** (the contract comes first; the implementation obeys the contract):

```mermaid
flowchart LR
  Types[".types.ts<br/>contract"] --> Styles["tv()<br/>variants"] --> Component[".tsx"] --> Docs[".stories.tsx<br/>+ .test.tsx"] --> Barrel["index.ts"]
```

**Recurring decisions (always question them):**

- Need a different **look** → a new `variant`/`size` in the component's `tv()` — never a class coming from outside.
- Need a different **behavior** → a semantic prop; research what Mantine/Ant Design/Chakra call the same concept before inventing a name.
- The **app** wants to customize → theme (design tokens selected in the root Provider), decided once — never per instance.
- Several elements with **shared state** → compound component with internal context.

## Governed by the constitution

These laws live in `turystack-architecture-pattern` and are not restated in
this skill. They are what a primitive inherits from the architecture:

| ID | Law | How a primitive expresses it |
|---|---|---|
| `ARC-CTR-1` | A contract is declared once; downstream types derive from it. | `.types.ts` is the contract; a consumer derives with `Omit`/`Pick`, never a parallel type |
| `ARC-LAY-5` | The barrel exposes the public surface. | `index.ts` exports component + types; nobody imports the internal file |
| `ARC-LAY-6` | Organization by unit, no folder of technical type. | one folder per component; no global `styles/` or `helpers/` |
| `ARC-ERR-9` | Unavailability is stated, never hidden. | a blocked control renders disabled with a reachable reason; it is never removed |
| `ARC-SEC-10` | Untrusted data is neutralized at the output point. | rich HTML only through a sanitizer, never a raw injection prop |

## Invariants (the most broken ones)

1. **`className`/`style` are never public props** — the whole visual resolve is internal, via `tv()`; app customization is a theme decision (tokens in the Provider), never an instance one.
2. **A prop dictates behavior** — semantic API (`variant`, `size`, `loading`, `block`), never a CSS passthrough.
3. **Every component = the complete file set for its context**: `.tsx` · `.types.ts` · `.test.tsx` · `index.ts` — and, in the library, `.stories.tsx` (the barrel exports component + types; named export, never default).
4. **Strongly typed contract**: every value domain is a named, exported union; controlled/uncontrolled as the `value`/`defaultValue`/`onChange` pair; `onChange` delivers the value, never the DOM event.
5. **An interactive element is never raw HTML** — always an accessible headless primitive.
6. **Before inventing a prop, read the real `.types.ts`**; capability missing → extend the primitive (library first), never work around it in the app.
