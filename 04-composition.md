# Composition — headless, compound, provider

**Rules defined here:** `CPS-1` · `CPS-2` · `CPS-3` · `CPS-4` · `CPS-5` ·
`CPS-L1` · `CPS-L2` · `CPS-L3` — the law is the *Invariants* table below;
every ❌ item cites the id it violates.

## 🌐 Generic pattern

### Concept

The primitive **styles and types**; what delivers accessibility (focus, keyboard, aria, portal, positioning) is a **headless primitive** underneath. Components with parts present themselves as **compound components** — the consumer composes the parts, state flows through internal context. The design system's global state (theme, color scheme) lives in a **single Provider** at the app root.

### Pattern

1. Interactive element → a wrap of a headless primitive with the component's variants; raw interactive HTML only when no equivalent primitive exists.
2. Component with parts (`Modal.Header`, `Tabs.Item`) → compound with dot notation; the parts share state through the component's internal context, invisible to the consumer.
3. Render polymorphism (a button that becomes a link) → `asChild`: the component projects its behavior onto the child, with no arbitrary-element prop.
4. A primitive knows no domain: zero imports of SDK, route or business entity — the domain wrapper lives in the app (see Consumption).

### Invariants

| ID | Law (one line) | Gate |
|---|---|---|
| CPS-1 | An interactive element is never raw HTML — always an accessible headless primitive; native HTML only where no equivalent primitive exists (e.g.: `input type="file"`). | `grit:no-raw-interactive` |
| CPS-2 | Component with parts = compound component with dot notation; state between the parts flows through the component's internal context, never through prop drilling demanded of the consumer. | `manual` |
| CPS-3 | Render polymorphism via `asChild` — never an `as`/`component` prop that accepts an arbitrary element and leaks control of the render. | `grit:no-as-prop` |
| CPS-4 | Theme, color scheme and the design system's other global preferences live in a single Provider at the root; no primitive reads global configuration from anywhere else. | `manual` |
| CPS-5 | A primitive is domain-agnostic: zero SDK/domain/route imports; it receives ready data through a generic prop. | `biome:noRestrictedImports` |
| CPS-L1 | Compound components via `Object.assign(Root, { Part })` — preserves tree shaking and the parts' typing. | `manual` |
| CPS-L2 | Internal context in `<name>.context.ts` in the component folder; the consuming hook throws a clear error when used outside the Root. | `manual` |
| CPS-L3 | The headless primitive's composition tree is followed **exactly** — each primitive has its strict structure of parts; never improvise the hierarchy. | `manual` |

## 🛠️ Project-specific

### Mechanisms

- **CPS-1** — headless primitives via `@base-ui/react`, one module per element; the wrap applies the component's `tv()` and translates the contract (PROP-7: public `onChange` → the primitive's callback):

```tsx
// switch.tsx
import { Switch as SwitchPrimitive } from '@base-ui/react/switch'

export function Switch({ checked, defaultChecked, disabled, onChange }: SwitchProps) {
  return (
    <SwitchPrimitive.Root
      checked={checked}
      className={styles()}
      defaultChecked={defaultChecked}
      disabled={disabled}
      onCheckedChange={onChange}
    >
      <SwitchPrimitive.Thumb className={thumbStyles()} />
    </SwitchPrimitive.Root>
  )
}
```

- **CPS-L3** — the primitives' trees are strict; use the exact parts:

```tsx
// Dialog: Portal > Backdrop > Popup (not "Overlay"/"Content")
<Dialog.Root open={open} onOpenChange={onChange}>
  <Dialog.Portal>
    <Dialog.Backdrop className={backdropStyles()} />
    <Dialog.Popup className={popupStyles()}>{children}</Dialog.Popup>
  </Dialog.Portal>
</Dialog.Root>

// Menu and Popover: Portal > Positioner > Popup
```

- **CPS-2/L1** — compound with `Object.assign`, nested parts included:

```tsx
// modal.tsx
const Modal = Object.assign(ModalRoot, {
  Body: ModalBody,
  Footer: ModalFooter,
  Header: Object.assign(ModalHeader, {
    Description: ModalHeaderDescription,
    Title: ModalHeaderTitle,
  }),
})

export { Modal }
```

- **CPS-L2** — internal context placed in the folder (e.g.: `layout.context.ts` sharing state between `Layout.Sidebar` and `Layout.Content`):

```ts
// layout.context.ts
export function useLayoutContext() {
  const context = useContext(LayoutContext)
  if (!context) {
    throw new Error('Layout parts must be used inside <Layout>')
  }
  return context
}
```

- **CPS-3** — `asChild` in the contract; internally the component projects its behavior onto the child via the primitive's `render` (or `Slot`).

- **CPS-4** — the library's `Provider` at the app root: applies `theme` (via `data-theme`) and `defaultColorScheme`, persisting the user's preference.

### ✅ How to do it

```tsx
<Modal open={opened} onChange={setOpened}>
  <Modal.Header>
    <Modal.Header.Title>Confirm</Modal.Header.Title>
    <Modal.Header.Description>This action is permanent</Modal.Header.Description>
  </Modal.Header>
  <Modal.Body>{children}</Modal.Body>
  <Modal.Footer>
    <Button variant="outline" onClick={close}>Cancel</Button>
    <Button variant="destructive" onClick={confirm}>Delete</Button>
  </Modal.Footer>
</Modal>
```

```tsx
// a button that navigates: Button's behavior, Link's render
<Button asChild variant="link">
  <Link to="/settings">Settings</Link>
</Button>
```

### ❌ Never do

```tsx
// ❌ CPS-1 — raw interactive HTML when a primitive exists
<div role="dialog" onKeyDown={handleEscape}>...</div>

// ❌ CPS-3 — arbitrary-element prop
<Button component="a" href="/settings" />

// ❌ CPS-5 — primitive importing domain
import { useListUsers } from '@/~sdk/users'

// ❌ CPS-2 — demanding that the consumer do the internal wiring
<Modal open={opened} bodyProps={{ isOpen: opened }} />

// ❌ CPS-L3 — improvising the primitive's hierarchy
<Dialog.Root><Dialog.Popup>{children}</Dialog.Popup></Dialog.Root>
```
