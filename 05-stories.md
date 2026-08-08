# Stories — the contract's living doc

> **Scope:** stories live in the **shared library** — the app does not set up Storybook. The living doc of an app-local primitive is its `.types.ts` + real usage in the routes (see STB-4).

## 🌐 Generic pattern

### Concept

A story is the contract's **executable documentation**: every meaningful state of the component becomes a named example, renderable and visually inspectable. Whoever is going to use the component reads the stories before the types; whoever reviews a new component demands one story per variation the contract promises.

### Pattern

1. One named story per meaningful state: the canonical one (`Default`) + one per `variant`, per state (`Loading`, `Disabled`), per mode and per relevant section.
2. A story is self-contained: only `args` (and `render` when it needs local state) — no hand-configured controls/knobs.
3. Stories group by the same grouping as the barrel (Components, Form, Layout, Feedback...).

### Invariants

| id | Invariant |
|---|---|
| STB-1 | Every variation the contract promises (each `variant` value, each state boolean, each `mode`) has a named story — a contract without a story is an undocumented contract. |
| STB-2 | A story is a self-contained example via `args`; never manual configuration of controls/knobs (`argTypes`) — the example is the doc, not the playground. |
| STB-3 | The story group's title mirrors the component's category in the barrel — the doc navigation and the public API tell the same structure. |
| STB-4 | Stories live in the shared library; an app-local primitive that grows to the point of deserving stories (generic variants, reuse across domains) is a sign of **promotion to the library** — promote first, write the stories there. |
| STB-L1 | `Meta`/`StoryObj` with `satisfies`; the `<name>.stories.tsx` file placed in the component folder and excluded from the library build. |

## 🛠️ Project-specific

### Mechanisms

- **STB-L1** — standard skeleton (`@storybook/react-vite`):

```tsx
// button.stories.tsx
import type { Meta, StoryObj } from '@storybook/react-vite'

import { Button } from './button'

const meta = {
  args: {
    children: 'Button',
  },
  component: Button,
  title: 'Components/Button',
} satisfies Meta<typeof Button>

export default meta
type Story = StoryObj<typeof meta>

export const Default: Story = {}

export const Outline: Story = {
  args: {
    variant: 'outline',
  },
}

export const Loading: Story = {
  args: {
    loading: true,
  },
}
```

- **STB-1** — a component with state (modal, select) uses `render` with a local hook:

```tsx
export const Controlled: Story = {
  render: () => {
    const [value, setValue] = useState<string | null>(null)
    return (
      <Select
        mode="single"
        onChange={setValue}
        optionLabel="name"
        optionValue="id"
        options={fruits}
        value={value}
      />
    )
  },
}
```

### ✅ How to do it

- One named export per variation: `Default`, `Outline`, `Ghost`, `Destructive`, `Loading`, `Disabled`, `Block`, `WithLeftSection`.
- `title` in the barrel's categories: `Components/...`, `Form/...`, `Layout/...`, `Feedback/...`.

### ❌ Never do

```tsx
// ❌ STB-2 — manual knobs instead of examples
const meta = {
  argTypes: {
    variant: { control: 'select', options: ['default', 'ghost'] },
  },
}

// ❌ STB-1 — only Default when the contract promises 7 variants
```
