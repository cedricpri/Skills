# Vue / frontend security checklist

- `v-html`, `innerHTML` or third-party components rendering HTML with any user-controlled data. Sanitize (DOMPurify) or render as text.
- User data in `:href` / `:src` (`javascript:` URLs), dynamic `<component :is>` from user input.
- Secrets in `VITE_*` (or `VUE_APP_*`) variables or anywhere in the bundle: everything in the frontend is public. Check the built files for keys.
- Auth tokens in `localStorage` (readable by any XSS); prefer `httpOnly` cookies where the backend supports it.
- Authorization and premium checks done only in the frontend (route guards, hidden buttons) without the backend enforcing them.
- Sensitive data in the URL (tokens, emails) ending up in logs, history and `Referer` headers.
- Open redirects: `?redirect=` parameters used without checking they are relative or allowlisted.
- `postMessage` handlers that don't check `event.origin`.
- Third-party scripts (ads, analytics, widgets) loaded without integrity or from many origins; what data they can read.
- Source maps published in production exposing source code (low risk, but check it is intended).
- Service worker caching authenticated API responses that could be shown to the next user of the device.
- `npm audit --omit=dev` findings for runtime packages.
