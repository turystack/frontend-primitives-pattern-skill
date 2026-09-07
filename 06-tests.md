# Tests — the tested contract

**Rules defined here:** `TST-1` · `TST-2` · `TST-3` · `TST-4` · `TST-5` ·
`TST-6` · `TST-L1` — the law is the *Invariants* table below; every ❌ item
cites the id it violates.

## 🌐 Generic pattern

### Concept

A primitive's test covers the **observable contract**: what each prop promises happens on screen and in the interaction. It renders the way the user sees (by role and accessible name), interacts the way the user interacts (click, typing, keyboard) and asserts the promised effect. Implementation (CSS class, internal DOM structure) stays out — refactoring the look must not break a test.

### Pattern

1. Query by semantics: role + accessible name first; `data-testid` only when there is no semantics.
2. Interaction through real user events (click, typing), never a synthetic handler dispatch.
3. One test block per contract promise: each state boolean, the controlled/uncontrolled pair, the value delivered in `onChange`, each mode.

### Invariants

| ID | Law (one line) | Gate |
|---|---|---|
| TST-1 | A test asserts behavior observable through the prop contract — never an implementation detail (class string, internal tag, DOM order). Visual variation, when it needs an assert, is checked by a semantic attribute (role, `aria-*`, `data-*`). | `grit:no-implementation-query` |
| TST-2 | Every state boolean has a test of the compound effect: `loading` shows an indicator **and** blocks interaction; `disabled` blocks `onClick`. | `test:compound-state` |
| TST-3 | `onChange` is asserted by the **value** delivered (the domain), not by the call itself. | `manual` |
| TST-4 | A component with `mode`/a discriminated union tests each mode — including the type of the value delivered in each one. | `manual` |
| TST-5 | The component's pure util (`.utils.ts`) has its own test, isolated from the render. | `gate:util-tested` |
| TST-6 | Interaction in the test uses real user events (click, typing, keyboard) — never a synthetic handler/event dispatch. | `grit:no-synthetic-event` |
| TST-L1 | Vitest + Testing Library + `user-event`; the `<name>.test.tsx` file placed in the component folder. | `gate:test-file-placement` |

## 🛠️ Project-specific

### Mechanisms

- **TST-L1/1/2/3** — the `Button` test pattern:

```tsx
// button.test.tsx
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { describe, expect, it, vi } from 'vitest'

import { Button } from './button'

describe('Button', () => {
  it('fires onClick', async () => {
    const onClick = vi.fn()
    render(<Button onClick={onClick}>Save</Button>)

    await userEvent.click(screen.getByRole('button', { name: 'Save' }))

    expect(onClick).toHaveBeenCalledTimes(1)
  })

  it('blocks interaction while loading', async () => {
    const onClick = vi.fn()
    render(
      <Button loading onClick={onClick}>
        Save
      </Button>,
    )

    const button = screen.getByRole('button')
    expect(button).toBeDisabled()

    await userEvent.click(button)
    expect(onClick).not.toHaveBeenCalled()
  })
})
```

- **TST-3/4** — the domain value in the assert:

```tsx
it('delivers the selected value', async () => {
  const onChange = vi.fn()
  render(
    <Select
      mode="single"
      onChange={onChange}
      optionLabel="name"
      optionValue="id"
      options={[{ id: 'a1', name: 'Apple' }]}
    />,
  )

  await userEvent.click(screen.getByRole('combobox'))
  await userEvent.click(screen.getByRole('option', { name: 'Apple' }))

  expect(onChange).toHaveBeenCalledWith('a1')
})
```

### ✅ How to do it

- `screen.getByRole('button', { name: 'Save' })` — semantics first.
- `data-testid` on the root when the component has no natural role (e.g.: `Skeleton`).
- Controlled/uncontrolled pair: one test with a fixed `value` (does not change on its own) and one with `defaultValue` (changes internally).

### ❌ Never do

```tsx
// ❌ TST-1 — asserting a CSS class
expect(button.className).toContain('bg-primary')

// ❌ TST-1 — asserting internal structure
expect(container.querySelector('div > span > svg')).toBeTruthy()

// ❌ TST-3 — asserting only the call, without the value
expect(onChange).toHaveBeenCalled()

// ❌ TST-6 — synthetic interaction instead of user-event
fireEvent.click(button)
```
