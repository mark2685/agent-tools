# Accessibility (WCAG 2.1 AA)

Accessibility rules that apply when writing any user-facing component.

## Semantic HTML

- Use `<Button>` from `@hadrian-mtv/ui-toolkit/button` for actions — never `<div onClick>`
- Use `FlowLink`/`FlowLinkButton` from `@hadrian-mtv/flow-navigation` for navigation
- Use `<ul>`/`<ol>` with `role="list"` for lists of items
- Use heading levels (`h1`–`h6`) in order — don't skip levels
- Use `<label htmlFor>` for all form inputs, or `aria-label` when no visible label exists

## Keyboard Navigation

- Every interactive element must be reachable via Tab
- Custom interactive elements need `tabIndex={0}`, `role`, and `onKeyDown` handlers
- Escape should close dialogs, popovers, and dropdowns
- Enter/Space should activate buttons and toggleable controls

## Focus Management

- `Dialog` and `Sheet` from `@hadrian-mtv/ui-toolkit` use Radix primitives that handle focus trapping automatically — use these rather than building custom modals
- When building custom interactive overlays outside of Radix, move focus to the first interactive element on open and return focus to the trigger on close

## ARIA

- Icon-only `<Button variant="ghost" size="icon">` needs `aria-label` describing the action (e.g., `aria-label="Close dialog"`)
- Use `aria-busy="true"` on containers with loading skeletons
- Use `aria-live="polite"` for status messages that appear asynchronously (e.g., inline feedback after a mutation)
- Use `aria-expanded` on toggles that show/hide content
- Don't add ARIA attributes that duplicate native semantics (e.g., `role="button"` on `<Button>`)

## Color and Contrast

- Never use color as the sole indicator of state — pair with icons, text, or patterns
- Use design tokens (`surface-*`, `content-*`, `border-*`) which are designed for sufficient contrast
- Ensure 4.5:1 contrast ratio for normal text, 3:1 for large text (18px+ bold or 24px+)
- Status colors (`surface-positive`, `surface-negative`, `surface-warning`) must always be paired with a text label or icon

## Loading and Empty States

- Skeleton loaders should have `aria-busy="true"` and `aria-label` describing what's loading
- Empty states should be announced — use `role="status"` on the empty state container
- Error states should be focusable or announced via `aria-live`
