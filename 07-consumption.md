# Consumption — using and extending primitives

**Rules defined here:** `USO-1` · `USO-2` · `USO-3` · `USO-4` · `USO-5` ·
`USO-6` — the law is the *Invariants* table below; every ❌ item cites the id
it violates.

## 🌐 Generic pattern

### Concept

The consumer of a primitive works **against the real contract**, never against memory. Before using it: read the barrel (what exists) and the `.types.ts` (what each prop promises). When a capability is missing, the path is to **extend the primitive at the source** — the library first — and consume it afterwards; a visual workaround in the app does not exist, because there is no `className` to work around with. A domain wrapper derives its contract from the primitive; a product-specific primitive is born in `src/ui/` with the same grammar as this skill.

### Pattern

1. **Discovery before anything else**: barrel → `.types.ts` → story → implementation, in that order, until the doubt dies.
2. **Domain wrapper** (e.g.: a user select): derives its props from the primitive's exported type with `Omit` of the props it manages; passes the rest through intact.
3. **Missing capability**: new prop/variant on the primitive (library) → the library's typecheck/lint/build → consume it in the app. Never recreate the look in the app.
4. **Product-specific primitive**: `src/ui/<name>/` with the same 5 files, the same law for props and styles.

### Invariants

| ID | Law (one line) | Gate |
|---|---|---|
| USO-1 | Before using or extending: read the barrel and the real `.types.ts` — never invent a prop/value from memory; a prop that does not exist in the contract is a review reprove. | `manual` |
| USO-2 | A domain wrapper derives its contract from the primitive (`Omit` of the managed props) — never retypes a parallel contract by hand. | `gate:no-parallel-contract` |
| USO-3 | Missing capability → extend the primitive at the source (library first, `src/ui/` if specific) and consume it afterwards — never rebuild with HTML/CSS in the app what the primitive should offer. | `manual` |
| USO-4 | A product-specific primitive lives in the app's `src/ui/` and follows this skill **in full** (the app-local context's structure — no `.stories.tsx`, see STB-4 —, semantic props, internal variants, no `className`). | `manual` |
| USO-5 | Never duplicate an existing primitive — neither by copying the folder, nor by recreating it with raw HTML; a UI concept has exactly one owner. | `gate:no-duplicate-primitive` |
| USO-6 | Changed a primitive → update contract, stories, tests and barrel together, and validate the package (typecheck · lint · build) before consuming the change. | `manual` |

## 🛠️ Project-specific

### Mechanisms

- **USO-1** — discovery in seconds:

```sh
rg "export" src/index.ts                             # what the library exports
rg --files src/components | rg 'types.ts$'           # every contract
rg -n "loading|variant|size|mode|onChange" src/components -g '*.types.ts'
```

- **USO-2** — domain wrapper deriving from `SelectProps`:

```tsx
// components/users/user-select.tsx (in the app, next to the domain)
import { Select, type SelectProps } from '@turystack/react-web'

type UserSelectProps<K extends 'single' | 'multiple'> = Omit<
  SelectProps<User, string, string, K>,
  'loading' | 'onSearchChange' | 'optionLabel' | 'optionValue' | 'options' | 'searchable'
>

export function UserSelect<K extends 'single' | 'multiple'>(
  props: UserSelectProps<K>,
) {
  const [search, setSearch] = useState('')
  const users = useListUsers({ search })

  return (
    <Select
      {...props}
      loading={users.isLoading}
      onSearchChange={setSearch}
      optionLabel="name"
      optionValue="id"
      options={users.data ?? []}
      searchable
    />
  )
}
```

- **USO-4** — product primitive in `src/ui/`, same grammar:

```
src/ui/kanban-column/
├── kanban-column.tsx
├── kanban-column.types.ts
├── kanban-column.test.tsx
└── index.ts
```

- **USO-6** — validating the library before consuming:

```sh
pnpm typecheck && pnpm check && pnpm build
```

### ✅ How to do it

- `Button` missing `variant="link-muted"`? Add the union in `.types.ts`, the variant in the `tv()`, the story, the test — in the library — and only then use it in the app.
- The wrapper keeps the base props (`value`, `onChange`, `placeholder`, `disabled`, `size`, `clearable`) inherited from the primitive — the wrapper's consumer loses no API.

### ❌ Never do

```tsx
// ❌ USO-3 — rebuilding in the app what the primitive should offer
<div className="rounded-md border px-3">
  <input className="outline-none" />
</div>

// ❌ USO-2 — parallel contract typed by hand
type UserSelectProps = {
  value?: string
  onChange?: (value: string) => void
  placeholder?: string
}

// ❌ USO-5 — local copy of an existing primitive
src/ui/button/button.tsx

// ❌ USO-1 — prop invented from memory
<Button color="red" fullWidth />
```
