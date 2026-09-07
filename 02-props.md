# Props — the primitive's API

**Rules defined here:** `PROP-1` · `PROP-2` · `PROP-3` · `PROP-4` · `PROP-5` ·
`PROP-6` · `PROP-7` · `PROP-8` · `PROP-9` · `PROP-10` · `PROP-11` — the law is
the *Invariants* table below; every ❌ item cites the id it violates.

## 🌐 Generic pattern

### Concept

The prop **is** the design system's API: it dictates the component's behavior, and the component works out on its own how that turns into UI. The consumer declares **intent** (`variant="destructive"`, `loading`, `mode="multiple"`) — never appearance (class, color, padding). Every new prop is an API decision: think the way the big design systems (Mantine, Ant Design, Chakra) think — established name, enumerated value, sensible default.

### Pattern

A primitive's contract is composed of these blocks, always in this logic:

1. **Named unions** for every value domain (`ButtonVariant`, `ButtonSize`) — exported, reusable by wrappers.
2. **State booleans** (`loading`, `disabled`, `block`) — each with a well-defined compound effect (loading also disables).
3. **Sections** for adjacent content (`leftSection`, `rightSection`) — the consumer passes the node, the component resolves position and spacing.
4. **Controlled/uncontrolled pair** (`value` + `defaultValue` + `onChange`) for every component with a value.
5. **Mutually exclusive modes** as a discriminated union — the type forces the correct props for each mode.
6. **Generics** in data components — the option type flows from the consumer, with extractors for label/value.

### Invariants

| ID | Law (one line) | Gate |
|---|---|---|
| PROP-1 | A prop is semantic and dictates behavior; **no** prop passes CSS through — there is no `className`, `style`, free-form color, numeric spacing or any arbitrary visual value in the public API. | `grit:no-classname-prop` |
| PROP-2 | Every value domain is a **named and exported** union (`ButtonVariant = 'default' \| 'ghost' \| ...`), never an inline union on the prop. | `grit:no-inline-union` |
| PROP-3 | Before naming a new prop, check what Mantine/Ant Design/Chakra call the same concept; only invent a name with no established precedent. | `manual` |
| PROP-4 | Interactive state via booleans with a defined compound effect: `loading` shows an indicator **and** blocks interaction; `disabled` blocks; `block` takes up the full width. | `test:compound-state` |
| PROP-5 | Adjacent content comes in through sections (`leftSection`/`rightSection`); the component owns the layout between section and content. | `manual` |
| PROP-6 | A component with a value exposes the pair `value` (controlled) + `defaultValue` (uncontrolled) + `onChange` — never just one of the sides. | `gate:controlled-pair` |
| PROP-7 | `onChange` delivers the **domain value** (string, number, `T[]`, `null`), never the platform's raw event. | `grit:onchange-delivers-value` |
| PROP-8 | Mutually exclusive modes = discriminated union on the `mode` prop; the type of `value`/`onChange` changes with the mode and the compiler rejects an invalid combination. | `manual` |
| PROP-9 | A data component is generic (`<T, I, O>`): `options: T[]` + `optionLabel`/`optionValue` extractors as `keyof T` or a function — the consumer never pre-maps data into a component-specific shape. | `manual` |
| PROP-10 | Every optional behavior prop has a default declared in the component (destructuring or `defaultVariants`) — a consumer who passes nothing gets the canonical primitive. | `manual` |
| PROP-11 | An icon enters as a **node slot** (`leftSection`, `icon`), never as a name string the component resolves. Icons come from `@turystack/react-icons`; the primitive sizes and hides them (`AXS-L1`) and never ships an icon registry of its own. | `grit:no-icon-name-prop` |

## 🛠️ Project-specific

### Mechanisms

- **PROP-1/2/4/5** — canonical `Button` contract (no `className`, no `style`; `ariaLabel` gives the icon-only button an accessible name):

```ts
// button.types.ts
export type ButtonType = 'button' | 'submit' | 'reset'

export type ButtonSize = 'sm' | 'md' | 'lg' | 'icon-sm' | 'icon-md' | 'icon-lg'

export type ButtonVariant =
  | 'default'
  | 'destructive'
  | 'outline'
  | 'dashed'
  | 'secondary'
  | 'ghost'
  | 'link'

export type ButtonProps = {
  ariaLabel?: string
  form?: string
  type?: ButtonType
  size?: ButtonSize
  variant?: ButtonVariant
  leftSection?: React.ReactNode
  rightSection?: React.ReactNode
  block?: boolean
  loading?: boolean
  disabled?: boolean
  asChild?: boolean
  onClick?: React.MouseEventHandler<HTMLButtonElement>
}
```

- **PROP-6/7/8/9** — `Select` with modes and generics; `K extends SelectMode` selects the contract:

```ts
// select.types.ts
export type SelectMode = 'single' | 'multiple'

export type BaseSelectProps<T, O> = {
  options: T[]
  optionLabel: keyof T | ((option: T) => string)
  optionValue: keyof T | ((option: T) => O)
  renderOption?: (option: T) => React.ReactNode
  placeholder?: string
  searchable?: boolean
  clearable?: boolean
  disabled?: boolean
  loading?: boolean
  size?: SelectSize
}

export type SelectSingleProps<T, I = string, O = I> = BaseSelectProps<T, O> & {
  mode: 'single'
  value?: I | null
  defaultValue?: I | null
  onChange?: (value: O | null) => void
}

export type SelectMultipleProps<T, I = string, O = I> = BaseSelectProps<T, O> & {
  mode: 'multiple'
  value?: I[]
  defaultValue?: I[]
  onChange?: (value: O[]) => void
}

export type SelectProps<
  T,
  I = string,
  O = I,
  K extends SelectMode = SelectMode,
> = K extends 'single'
  ? SelectSingleProps<T, I, O>
  : K extends 'multiple'
    ? SelectMultipleProps<T, I, O>
    : never
```

- **PROP-10** — visual defaults live in the `tv()`'s `defaultVariants` (see Styles); behavior defaults in the destructuring:

```tsx
function Button({ type = 'button', ...props }: ButtonProps) {}
```

### ✅ How to do it

```tsx
// intent, not appearance
<Button variant="destructive" size="sm" loading leftSection={<Trash2 />}>
  Delete
</Button>

// onChange delivers the value
<Select
  mode="multiple"
  options={users}
  optionLabel="name"
  optionValue="id"
  value={selected}
  onChange={setSelected}
/>
```

### ❌ Never do

```tsx
// ❌ PROP-1 — CSS passthrough in the public API
<Button className="bg-red-500" style={{ padding: 8 }} />

// ❌ PROP-2 — inline union, no exported name
type Props = { variant?: 'default' | 'ghost' }

// ❌ PROP-7 — onChange leaking the DOM event
onChange?: (event: React.ChangeEvent<HTMLInputElement>) => void

// ❌ PROP-8 — mode as a loose boolean instead of a discriminated union
type Props = { multiple?: boolean; value?: string | string[] }

// ❌ PROP-9 — forcing the consumer to pre-map the data
options: Array<{ label: string; value: string }>
```
