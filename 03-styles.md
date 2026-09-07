# Styles — internal variants, zero className

**Rules defined here:** `STY-1` · `STY-2` · `STY-3` · `STY-4` · `STY-5` ·
`STY-6` · `STY-7` · `STY-8` · `STY-9` · `STY-10` · `STY-L1` · `STY-L2` · `STY-L4` ·
`STY-L3` — the law is the *Invariants* table below; every ❌ item cites the id
it violates.

## 🌐 Generic pattern

### Concept

The component is the **absolute owner of its look**. Every style is resolved internally by a declarative variant system: base + named variations + defaults. The public API knows only the semantic variants (`variant`, `size`) — class, color and spacing never cross the component's boundary, in either direction. When an **app** needs its own identity, that is a **theme** decision — design tokens selected once in the root Provider — never an instance decision.

### Pattern

1. One variant object per styleable element: the root element's + one per sub-element.
2. Each object declares `base` (what never changes), `variants` (each axis of variation), `defaultVariants` (the canonical one) and `compoundVariants` (intersections).
3. The variant keys mirror the contract's unions 1:1 — the type in `.types.ts` and the `tv()` tell the same story.
4. Every color/surface references the theme's **semantic tokens** (`primary`, `muted`, `destructive`...) — swapping the theme swaps the whole identity without touching a component.

### Invariants

| ID | Law (one line) | Gate |
|---|---|---|
| STY-1 | `className`/`style` never come in or go out through the component's public API — the visual resolve is 100% internal. | `grit:no-classname-prop` |
| STY-2 | Every style is declared in the variant system (base/variants/defaultVariants) — never a loose conditional class in the render (string concatenation, class ternary, inline style). | `grit:no-conditional-class` |
| STY-3 | The contract's `variant`/`size` unions and the variant keys in the style are 1:1 — adding a value to the union without the matching variant (or vice versa) is a reprove. | `gate:variant-union-parity` |
| STY-4 | Every axis of variation with a default declares `defaultVariants` — the component with no props renders the canonical one. | `gate:defaults-declared` |
| STY-5 | Each styleable sub-element has its own variant object; the root element's is the file's main one. | `manual` |
| STY-6 | Intersection of axes (e.g.: `variant` ghost + `size` icon) → `compoundVariants`, never class logic in the render. | `manual` |
| STY-7 | Dimension and spacing exposed in the API follow a token scale (`sm`/`md`/`lg`, `none`/`xs`/`sm`/`md`/`lg`/`xl`) — never a numeric/arbitrary value. | `manual` |
| STY-8 | Per-app visual identity = **theme**: design tokens (semantic CSS variables) selected in the root Provider; the component references the token (`primary`, `muted`), never a hardcoded brand value — per-instance customization does not exist. | `grit:no-hardcoded-brand` |
| STY-9 | Light and dark are resolved by the **tokens**, never by the component: no primitive branches on the color scheme, and no variant exists in a `dark` flavour. A scheme the component can see is a scheme it can get wrong in one place and right in another. | `grit:no-scheme-branch` |
| STY-10 | The token set is the contract between library and app: the app chooses **values**, the library chooses **which tokens exist**. An app that needs a token the library does not publish has found a gap to close upstream, not a place for a local variable. | `manual` |
| STY-L1 | Styles declared at the **top of the component's file** with `tv` (tailwind-variants); the root element's object is exported under the name `styles`. | `gate:styles-export` |
| STY-L2 | A style shared between components lives in the owner component's `<name>.shared.ts` and is imported by the others — never copied. | `manual` |
| STY-L3 | `cn()` (clsx + tailwind-merge, in `support/utils`) only for **internal** conditional merging — never to accept a class coming from outside. | `grit:no-external-class-merge` |
| STY-L4 | Every element the component paints carries a stable semantic class beside its utilities, derived from the slot's key (`cellContent` → `<name>-cell-content`; a `base` or `root` → `<name>-root` or the bare `<name>`). It is the surface a theme reaches for: an element without one renders correctly and cannot be restyled. | `gate:slots-named` |

## 🛠️ Project-specific

### Mechanisms

- **STY-1..6/L1** — the `Button` pattern: root exported as `styles`, sub-element with its own tv:

```ts
// button.tsx (top of the file)
import { tv } from 'tailwind-variants'

const contentStyles = tv({
  base: 'inline-flex items-center gap-2',
  variants: {
    loading: {
      true: 'invisible',
    },
  },
})

export const styles = tv({
  base: 'relative inline-flex cursor-pointer items-center justify-center gap-2 rounded-md font-medium text-sm transition-colors disabled:pointer-events-none disabled:opacity-50',
  defaultVariants: {
    size: 'md',
    variant: 'default',
  },
  variants: {
    block: {
      true: 'w-full',
    },
    size: {
      lg: 'h-11 px-8',
      md: 'h-10 px-4 py-2',
      sm: 'h-9 px-3',
    },
    variant: {
      default: 'bg-primary text-primary-foreground hover:bg-primary/90',
      ghost: 'hover:bg-accent hover:text-accent-foreground',
      outline: 'border border-input bg-input/30 hover:bg-accent',
    },
  },
})
```

- **STY-8** — themes are CSS files of semantic variables (`--background`, `--primary`, `--radius`...), selected via the `data-theme` attribute on `<html>` and dark mode via the `.dark` class; the Provider applies and persists the choice. The component only uses the token's utility class (`bg-primary`) — never the color behind it.

- **STY-L2** — `button.shared.ts` when another component (e.g.: a menu trigger that looks like a button) needs the same look: the `tv` lives there and both import it.

- **STY-L3** — `cn()` merges the `tv()` result with internal conditional classes that are not an axis of variation; it never receives a class from a prop.

### ✅ How to do it

```ts
// new visual need = a new declared variant (union + tv, 1:1)
variant: {
  'link-muted': 'text-foreground underline underline-offset-4',
}
```

```ts
// intersection via compoundVariants
compoundVariants: [
  {
    class: 'px-0',
    size: 'sm',
    variant: 'link',
  },
]
```

### ❌ Never do

```tsx
// ❌ STY-1 — accepting a class from outside
export type ButtonProps = { className?: string }

// ❌ STY-2 — conditional class in the render
<button className={`btn ${loading ? 'opacity-50' : ''}`} />

// ❌ STY-2 — inline style
<button style={{ width: block ? '100%' : undefined }} />

// ❌ STY-7 — arbitrary value in the API
<Flex gap={13} />

// ❌ STY-8 — hardcoded brand color instead of a semantic token
variant: {
  default: 'bg-[#7c3aed] text-white',
}
```
