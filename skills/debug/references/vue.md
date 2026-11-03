# Vue / frontend bug patterns

Check the ones that match the symptom. Confirm against the installed Vue version (`npm ls vue`).

## Reproduce in a real browser
- Use Playwright (or the browser devtools) with the same viewport as the report: a mobile bug at 390px wide often doesn't exist at 1280px.
- Record console errors and failed network requests while reproducing. A UI bug is often a failed or unexpected API response.
- Check the exact response the component received, not the one the API "should" return.

## Reactivity
- Destructuring a `reactive()` object, or `props` before Vue 3.5, loses reactivity. Use `toRefs`, `computed`, or access through the object.
- Replacing a `reactive()` object wholesale (`state = {...}`) breaks the reference held by the template.
- A `watch` on a ref holding an object only fires when the ref is replaced, not on nested changes, unless `deep: true`. A `watch` on a getter returning the same object never fires.
- A `computed` that mutates state, or depends on something non-reactive (`localStorage`, `Date.now()`, a plain module variable), shows stale values.
- Data fetched in `onMounted` without being re-fetched when a route param changes (same component reused between `/flights/1` and `/flights/2`).

## Rendering
- `v-for` with `:key="index"` mixes up component state, inputs and animations when items are inserted, removed or sorted. Use a stable id.
- `v-if` together with `v-for` on the same element: `v-if` has higher priority in Vue 3.
- An element rendered before its data arrives: missing loading and empty states, `undefined` in templates, layout jumps.
- Hydration or flash issues from values that differ between first render and after mount (dates, `window` width).

## Async and state
- Race conditions: two requests in flight and the slower, older one wins (typing in a search box, switching tabs fast). Cancel with `AbortController` or ignore stale responses.
- Errors swallowed in a `catch` that only logs, leaving the UI in a loading state forever.
- Shared state (Pinia store, module-level variable) not reset between pages, users or after logout.
- Form state not reset after submit or when the edited record changes.

## Layout, CSS and mobile
- Mobile spacing and overflow: fixed widths, `100vh` on mobile browsers (use `dvh`), long unbroken strings (codes, emails), flex children without `min-width: 0`.
- Safe areas and the on-screen keyboard on iOS and Android, sticky elements under the browser bar.
- Scoped styles not reaching child components (`:deep()`), Tailwind classes built dynamically so they are purged from the build.
- z-index and stacking contexts for overlays, modals and dropdowns.

## Build and environment
- Works in `npm run dev` but not in the build: environment variables (`import.meta.env.VITE_*` are baked in at build time), case-sensitive import paths, dynamic imports, tree-shaken side effects.
- Browser cache or service worker serving an old bundle after a deploy.
- Dates: `new Date("2026-03-01")` is parsed as UTC, `new Date("2026-03-01T10:00")` as local time; `toLocaleString` differs between browsers and locales.

## i18n
- Missing translation keys in one locale fall back silently or show the key. Check every locale file for the key.
- Pluralization and interpolation parameters that don't match between locales.
