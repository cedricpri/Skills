# Vue / frontend review checklist

## Components
- Follows the existing component style of the repo (Options vs Composition API, `<script setup>`, TypeScript or not).
- Props typed and validated, with defaults for optional ones; no mutation of props.
- Emits declared; no reaching into child or parent internals.
- Components small enough to read; no copy-pasted blocks that already exist as a component.

## Reactivity
- No destructuring of `reactive()` objects or of props (before Vue 3.5) that loses reactivity.
- `computed` without side effects; `watch` with the right `deep` / `immediate` options; watchers cleaned up when created outside setup.
- Data re-fetched when route params change for reused components.

## Async and state
- Loading, empty and error states for every request; errors shown to the user, not only logged.
- Buttons disabled during submit to prevent double submits.
- Stale responses ignored or requests cancelled when the input changes quickly.
- Store state reset on logout and when switching records.

## Rendering
- `v-for` keyed by a stable id, not the index.
- No `v-if` and `v-for` on the same element.
- Long lists paginated or virtualized.

## Layout and mobile
- Works at phone width (around 360-390px) and desktop: no horizontal overflow, readable text, tap targets large enough.
- Long values (names, codes, emails) wrap or truncate.
- Consistent with the existing design system or Tailwind conventions of the repo; no one-off colours and spacing values.

## Security
- No `v-html` with untrusted data.
- No secrets in `VITE_*` variables (they ship to the browser).
- Authorization not enforced only by hiding UI.

## i18n
- Every new string uses the i18n helper, with the key present in every locale file.
- Dates, numbers and units formatted for the locale.

## Accessibility basics
- Inputs have labels, buttons have accessible names, images have `alt`.
- Keyboard focus works for modals and dropdowns; sufficient contrast.

## Tests
- Component or E2E tests for new flows where the repo has them (vitest, Cypress, Playwright).
- Selectors that survive restyling (`data-test` attributes or roles), not CSS classes.
