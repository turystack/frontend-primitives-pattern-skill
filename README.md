# tury-stack-frontend-primitives-pattern — index

The grammar of how to write UI primitive components (shared library or the app's `src/ui/`). Start with `SKILL.md` (how to use it) and `00-overview.md` (mental model + invariants).

| Section | Ids | Covers |
|---|---|---|
| `00-overview.md` | — | Context (library vs app-local), mental model, most broken invariants |
| `01-component-structure.md` | `CMP-n` | Folder per component, 5 files, barrel, named exports |
| `02-props.md` | `PROP-n` | Semantic API, named unions, sections, controlled/uncontrolled, modes, generics |
| `03-styles.md` | `STY-n` | Internal tv(), zero className, defaultVariants, tokens + theme via Provider |
| `04-composition.md` | `CPS-n` | Headless primitives, compound components, asChild, single Provider |
| `05-stories.md` | `STB-n` | One story per meaningful state, args only — library only (app-local has no Storybook) |
| `06-tests.md` | `TST-n` | Observable contract, user-event, value in onChange |
| `07-consumption.md` | `USO-n` | Discovery, wrapper with Omit, extending the library, app-local primitive |
