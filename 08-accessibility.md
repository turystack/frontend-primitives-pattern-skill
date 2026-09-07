# Accessibility — the unit's obligations

**Concept.** Accessibility fails in the app, but it is **fixable in the
primitive**. A `Button` that has no channel for an accessible name produces one
violation per consumer; the same button with a required name prop produces zero,
forever. This section is the set of obligations a primitive carries so the app
cannot get it wrong — the app-side composition rules live in
`turystack-frontend-pattern` › `08-accessibility.md`.

**Rules defined here:** `AXS-1` · `AXS-2` · `AXS-3` · `AXS-4` · `AXS-5` ·
`AXS-6` · `AXS-L1` — the law is the *Invariants* table below; every ❌ item
cites the id it violates.

## 🌐 Generic pattern

**Concept.** Everything below follows from one idea: the primitive is the last
place a rule can be enforced by construction. Above it, every rule is a
convention someone has to remember.

**Pattern.** Each obligation is either a **type that refuses** the wrong usage,
or **behavior the component performs** so the consumer does not have to.

### Invariants

| ID | Law (one line) | Gate |
|---|---|---|
| AXS-1 | Every interactive primitive has a name channel, and an icon-only mode makes it **required by the type** — not by documentation. | `gate:name-channel` |
| AXS-2 | Every state the component shows visually is also exposed to assistive technology: `disabled`, `loading`, `selected`, `expanded`, `invalid`. A visual-only state does not exist for a screen reader. | `test:state-exposed` |
| AXS-3 | The focus indicator is part of the component's styles and is never removed. A component that hides focus to look cleaner is broken for keyboard users. | `grit:no-outline-none` |
| AXS-4 | Overlay behavior — focus trap, initial focus, restore on close, dismiss on Escape — is delegated to the headless primitive and never re-implemented by hand (`CPS-1`, `CPS-L3`). | `grit:no-manual-focus` |
| AXS-5 | When a primitive can be blocked with a reason, it provides the reachable channel for that reason itself: a disabled control takes no focus and fires no pointer event, so the component renders the focusable wrapper and binds the reason as the accessible description. | `test:denial-visible` |
| AXS-6 | Motion declared in the component's variants has a reduced-motion counterpart in the same place. | `gate:reduced-motion` |
| AXS-L1 | Icons rendered inside a primitive are decorative by default (`aria-hidden`); the accessible name comes from text or from the name prop, never from the icon. | `grit:no-icon-name-prop` |

**Why AXS-5 is the primitive's job and not the app's.** The app is told to keep
a blocked action visible and inert with its reason (`ARC-ERR-9`). It cannot
comply on its own: the control it was handed swallows pointer events and refuses
focus the moment it is disabled, so any tooltip the app attaches is unreachable
by keyboard and invisible to a screen reader. The wrapper has to come from
inside. A law that the consumer is structurally unable to satisfy is not a law,
it is a complaint.

## Governed by the constitution

These laws live in `turystack-architecture-pattern` and are not restated here.

| ID | Law | How a primitive expresses it |
|---|---|---|
| `ARC-ERR-9` | Unavailability is stated, never hidden. | `AXS-5`: the component supplies the channel the reason travels on |
| `ARC-CTR-1` | A contract is declared once. | the name requirement lives in `.types.ts`, so every consumer inherits it |

## 🛠️ Project-specific

**Mechanisms per rule:**

- **AXS-1** — the discriminated union does the work, so the wrong call does not
  compile:

```typescript
type ButtonProps =
  | { size?: Exclude<ButtonSize, IconSize>; children: ReactNode; label?: never }
  | { size: IconSize; children?: never; label: string }
```

  An icon-only button without a `label` is a type error, not a review comment.
  This is `PROP-8` (discriminated modes) used for an accessibility outcome.

- **AXS-2** — the same boolean that drives the variant drives the attribute.
  `loading` renders the indicator, blocks interaction **and** exposes a busy
  state; `invalid` styles the field **and** marks it invalid with its message
  associated. One prop, both consequences, decided inside (`PROP-4`).

- **AXS-3** — the focus ring is a variant in `tv()` like any other style
  (`STY-2`), built from theme tokens (`STY-8`). It is never removed, and it is
  never left to the browser default plus an `outline: none` somewhere else.

- **AXS-4** — `Modal`, `Sheet`, `DropdownMenu`, `Popover` and `Tooltip` follow
  the headless primitive's part tree exactly (`CPS-L3`). Manual focus
  management on top of it — `autoFocus` races, `element.focus()` in an effect —
  fights the trap and produces the bug it was trying to fix.

- **AXS-5** — the component renders the wrapper; the consumer passes a reason:

```tsx
<Button disabled={Boolean(blockedReason)} blockedReason={blockedReason}>
  Cancel order
</Button>
```

  Inside, the disabled branch wraps the control in a focusable element, hangs
  the tooltip off it, and binds the text as the control's accessible
  description. The consumer never assembles that by hand — which is exactly why
  `ARC-ERR-9` becomes achievable in the app.

- **AXS-6** — every animated variant carries its reduced-motion counterpart next
  to it, so the two can never drift apart.

- **AXS-L1** — icons come from `@turystack/react-icons`, are hidden from
  assistive technology inside the primitive, and never carry the name
  (`PROP-11`).

### ❌ Never do

```tsx
// ❌ [AXS-1] an icon-only mode that accepts no name
<Button size="icon-sm" leftSection={<TrashIcon />} />

// ❌ [AXS-2] a state that exists only as a class
<div className={loading ? 'opacity-50' : ''}> // nothing announces it

// ❌ [AXS-3] removing the focus ring for looks
'focus:outline-none' // with no replacement indicator

// ❌ [AXS-4] hand-rolled overlay behavior on top of the headless primitive
useEffect(() => { closeButtonRef.current?.focus() }, []) // fights the trap

// ❌ [AXS-5] a disabled control expected to host its own tooltip
<Tooltip content={reason}><Button disabled /></Tooltip> // never reachable

// ❌ [AXS-L1] the icon carrying the accessible name
<TrashIcon aria-label="Delete" /> // the name belongs to the control
```
