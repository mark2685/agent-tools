---
globs:
  - "apps/**/*.tsx"
  - "packages/**/*.tsx"
---

# Accessibility (WCAG 2.2 AA)

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

## WCAG 2.2 criteria

- **SC 2.4.7 Focus Visible (AA) / SC 2.4.11 Focus Not Obscured, Minimum (AA).** A visible keyboard-focus indicator must exist — never `outline: none` without a replacement; use the design-token focus ring — and it must not be entirely hidden by sticky headers, footers, or other author content.
- **SC 2.4.13 Focus Appearance (AAA).** Beyond mere visibility, the focus indicator should be at least 2px thick around the component's perimeter and meet a 3:1 contrast ratio between its focused and unfocused states. This is a Level **AAA** target, not an AA requirement — we adopt it as our house standard via the design-token focus ring, so do not suppress it.
- **SC 2.5.8 Target Size, Minimum (AA).** Pointer targets are at least 24×24px, or have 24px of spacing to neighboring targets. Icon buttons (`size="icon"`) already meet this — do not shrink them below 24px. Inline text links are exempt.
- **SC 2.5.7 Dragging Movements (AA).** Any drag operation (reorder, slider, drag-to-assign) needs a single-pointer alternative — buttons, a menu, or keyboard. Never make a drag the only way to perform an action.
- **SC 3.3.7 Redundant Entry (A).** Within one multi-step flow, do not ask the user to re-enter information they already provided. Auto-populate or offer a "same as" / select-previous control instead.

## Interactive elements

- Never nest interactive controls (a `<button>` inside a `<button>`, a `<Link>` wrapping a `<Button>`). It breaks keyboard and screen-reader semantics and is invalid HTML. Place sibling controls or split the affordances.
- A disabled control must convey *why* it is disabled — a tooltip, helper text, or inline message. A silently-disabled button gives the user no path forward.

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
