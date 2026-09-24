# Toast

**Purpose:** Non-blocking, auto-dismissing feedback for the result of an action — "Copied," "Added to Discovery," "Couldn't save." Distinct from a confirmation dialog, which blocks and requires an explicit choice.

## RESOLVED 2026-09-24: mobile gets a real toast system — this is a feature to build, not drift to reconcile

Mobile has **no toast system at all**. Every piece of feedback — success, error, or otherwise — goes through React Native's native `Alert.alert()`, a blocking modal dialog requiring an explicit tap to dismiss, across at least 29 files. Auditing those call sites: only a handful are genuinely confirm-or-cancel decisions that deserve a blocking dialog — "Delete post?", "Delete page?", "Delete bookmark?", "Delete circle?", "Deactivate invite?" (all pass a `[Cancel, Confirm]` button array to `Alert.alert`). **The rest — the large majority — are single-message, no-decision feedback**: "Copied," "Added to Discovery," "Couldn't save," "Couldn't add to Discovery," "Reported," "Blocked," "Muted," "Couldn't restore" (five separate sites), and more. Every one of those is exactly what web's `toast.success()`/`toast.error()` handles without interrupting the user.

**Contract:** build a toast system on mobile with the same API shape as web's (`toast.success(message, {detail, action, durationMs})` / `toast.error(...)` / `toast.info(...)` / `toast.dismiss(id)` / `toast.clear()`), and migrate every `Alert.alert()` call that isn't a real confirm-or-cancel decision over to it. `Alert.alert` stays, deliberately, for the destructive/confirmation cases — that's the correct tool for those, not something to also convert.

## Visual contract, ported from web's existing (good) design

- Left-edge accent bar colored by kind: `success` → `success` token, `info` → `primary` token, `error` → `error` token
- Icon (lucide `CheckCircle2`/`Info`/`AlertCircle`, matching the kind) + message + optional secondary detail line + optional action button/link + a dismiss `X`
- Auto-dismiss: success/info after 4s, error after 8s, `0` = sticky (stays until manually dismissed) — same on both platforms
- **Fix while porting: web's toast has a real drop shadow (`shadow-lg`)** — violates IDEOLOGY.md §2 rule 2 same as other components found this series. Remove on web; don't introduce it on mobile.

## Found while auditing: web's own positioning comment is stale

`ToastStack.jsx`'s file comment says toasts are *"positioned bottom-right on desktop and bottom-inset full-width on mobile web."* The actual CSS does the opposite of what "mobile" suggests: base classes (`top-2 inset-x-2`, i.e. narrow-viewport) place it at the **top**, full-width; only the `lg:` breakpoint moves it to bottom-right. Not a bug — top placement on a narrow viewport is a reasonable choice — just a stale comment, worth fixing during implementation.

**OPEN — needs a decision for the native app specifically:** should mobile's toast sit at the top (below the safe area) or bottom (above the tab bar)? Not deciding this here — the app has a persistent bottom tab bar and multiple bottom sheets already (`components/Modal.md`), which argues for top placement to avoid visual collision, but that's a recommendation, not a resolved call.

## Tokens used

- Palette: `success`, `primary` (info), `error` (bar + icon color), `base-100`/`base-300` (surface, border), `base-content` at various opacities (message, detail text)
- Typography: `font-ui` for message/labels (chrome — a toast is chrome, not reader-controlled content, even when it's reporting on content the reader just interacted with)
