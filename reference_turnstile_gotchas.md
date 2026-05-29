---
name: Cloudflare Turnstile integration gotchas
description: Non-obvious Cloudflare Turnstile behaviors that bite when integrating the widget — verified during the first-number build.
type: reference
originSessionId: d4c09cb0-55c1-4317-9bc3-559855edf48a
---

Hit while integrating Turnstile in a Next.js app (May 2026). Save future debugging time.

**`size: "invisible"` is deprecated in `turnstile.render()`.** The render-time `size` param only accepts `"compact"`, `"flexible"`, `"normal"` now. Invisible behavior is configured at the **widget level in the Cloudflare dashboard** (Widget Mode → Invisible). Passing `size: "invisible"` throws `TurnstileError`.

**`execution: "execute"` is required for manual-trigger pattern.** Without it, the widget auto-runs at render time and your subsequent `turnstile.execute(widgetId)` call fires the "already executing" warning. Pass `execution: "execute"` in `render()` opts if you want to defer the check until form-submit.

**Don't hide the widget container with `display: none`.** Even in Managed/Invisible widget modes, low-trust sessions (incognito, fresh browsers, VPNs) get a fallback challenge UI rendered into the container. If hidden, the user has no way to complete it → silent 403. Use `empty:hidden` (Tailwind) or similar so the container collapses only when the widget renders nothing.

**Widget Mode "Invisible" silently fails low-trust sessions.** No UI fallback. If user trust signals are weak, siteverify just returns `success: false`. Use **Managed** mode unless you have a strong reason — Managed is invisible to trusted users and shows a checkbox to others.

**Hostname allowlist is enforced.** Token from a hostname not registered in the Turnstile widget's hostname list will fail siteverify. Vercel preview URLs (`*-<hash>-<scope>.vercel.app`) are different hostnames from the production alias — add what you need.

**`NEXT_PUBLIC_TURNSTILE_SITE_KEY` must be present at BUILD time** on Vercel. `process.env.NEXT_PUBLIC_*` is inlined at build, not runtime. Set in Vercel project env BEFORE the build that needs it; otherwise client gets empty string and widget fails to render.

**Cloudflare iframe console noise is unfixable.** `TrustedHTML/Script/ScriptURL` CSP violations, `xr-spatial-tracking` permissions warnings, audio/video codec parse warnings, `postMessage` origin mismatch, `cmg/1` preload-not-used warnings — all originate inside `challenges.cloudflare.com` iframe internals. Filter DevTools console with `-challenges.cloudflare.com` to ignore.

**siteverify error codes worth knowing:**

- `timeout-or-duplicate` → token expired (~5 min) or already used (single-use enforcement)
- `invalid-input-secret` → server secret doesn't match the site key
- `invalid-input-response` → bogus/tampered token
- Add `console.error("[turnstile] verify failed", data["error-codes"])` in your verifier so prod logs distinguish config bugs from genuine CAPTCHA failures.

## Related

[[cloudflare-turnstile]] [[turnstile-gotchas]] [[next-build-env]]
