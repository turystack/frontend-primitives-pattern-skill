# Component Structure

**Rules defined here:** `CMP-1` · `CMP-2` · `CMP-3` · `CMP-4` · `CMP-5` ·
`CMP-L1` — the law is the *Invariants* table below; every ❌ item cites the id
it violates.

## 🌐 Generic pattern

### Concept

A primitive is **self-contained**: contract, style, behavior, living doc and test live together, in a single folder named after the component. Whoever looks at the folder sees the whole component; whoever looks at the barrel sees the whole public API.

### Pattern

```
<name>/
├── <name>.tsx           # implementation (imports the contract)
├── <name>.types.ts      # contract: props and named unions
├── <name>.stories.tsx   # living doc (library only — see 05-stories)
├── <name>.test.tsx      # tested contract: observable behavior
└── index.ts             # barrel: re-exports component + types
```

Auxiliary files go into the same folder whenever they exist: styles shared between components (`.shared.ts`), pure utilities (`.utils.ts` + `.utils.test.ts`), static data (`.data.ts`), internal context (`.context.ts`).

### Invariants

| ID | Law (one line) | Gate |
|---|---|---|
| CMP-1 | One component = one folder with implementation, contract (`types`), test and barrel; in the shared library, stories too. None is optional in its context. | `gate:component-files` |
| CMP-2 | The component barrel re-exports the component **and** every public type; the consumer imports only from the package's root barrel/`src/ui` — never a deep import of an internal file. | `gate:barrel-shape` · `gate:contract-sync` |
| CMP-3 | An auxiliary file (shared style, pure util, data, context) lives in the component folder and is prefixed with its name; a pure util carries its own test. | `gate:component-files` |
| CMP-4 | Folder and files in kebab-case with the component's name; the exported name is the PascalCase equivalent (`date-range-input/` → `DateRangeInput`). | `gate:kebab-case` |
| CMP-5 | The contract lives in its own file, separate from the implementation — the implementation **imports** the contract, never defines it inline. | `gate:component-files` |
| CMP-L1 | Named export always; `export default` never — not in the component, not in the barrel, not in the stories (except the `meta` the stories tool requires). | `grit:no-default-export` |

## 🛠️ Project-specific

### Mechanisms

- **CMP-1/CMP-4** — folder at `src/components/<name>/` (library) or `src/ui/<name>/` (app), all kebab-case:

```
src/components/button/
├── button.tsx
├── button.types.ts
├── button.stories.tsx
├── button.test.tsx
└── index.ts
```

- **CMP-2** — component barrel with a double re-export; the root barrel aggregates:

```ts
// button/index.ts
export * from './button'
export * from './button.types'
```

```ts
// src/index.ts (library) or src/ui/index.ts (app)
export * from './components/button'
export * from './components/card'
```

- **CMP-3** — examples: `button.shared.ts` (look shared with other triggers), `phone-input.utils.ts` + `phone-input.utils.test.ts` (pure util with its own test), `layout.context.ts` (internal compound context).

- **CMP-5** — the implementation opens by importing the contract:

```ts
// button.tsx
import type { ButtonProps } from './button.types'
```

### ✅ How to do it

```ts
// date-range-input/index.ts — barrel exports component + types, named
export * from './date-range-input'
export * from './date-range-input.types'
```

```tsx
// the consumer imports from the root barrel
import { Button, type ButtonProps } from '@turystack/react-web'
```

### ❌ Never do

```tsx
// ❌ CMP-2 — deep import of an internal file
import { Button } from '@turystack/react-web/dist/components/button/button'

// ❌ CMP-5 — inline contract in the implementation
export function Button(props: { variant?: 'default' | 'ghost' }) {}

// ❌ CMP-L1 — default export
export default Button

// ❌ CMP-1 — component without a test (or, in the library, without stories) — "I'll add it later" does not exist
```
