# Modal (web) / Sheet (mobile)

**Purpose:** Contextual content shown over the current screen — forms, confirmations, pickers. Two related but genuinely distinct patterns, not one component wearing two skins.

## The shape difference is a legitimate platform convention, not drift

Web uses a centered dialog (`Modal.jsx`: fixed-width box, translucent backdrop, header with title + text "Close" button). Mobile uses a bottom sheet (pinned to the bottom edge, `maxHeight` capped, slides/fades up from the bottom edge). Confirmed by reading a matched pair — `AddToDiscoveryModal.jsx` exists on both platforms, same feature, and independently arrived at these two different shapes. This is the same kind of legitimate difference as Field's underline treatment almost was, except here there's no reason to force convergence: a centered dialog makes sense on a wide desktop viewport, a bottom sheet is the native, thumb-reachable convention on a phone. **Contract: keep both shapes.** Web's primitive stays named `Modal`; mobile's gets a new shared primitive named `Sheet` — different names on purpose, since forcing one name onto two different interaction patterns would misrepresent them as the same thing.

## RESOLVED: mobile needs the shared primitive it doesn't have — this is Field/Heading's pattern, reversed

Web already has one clean, reused `Modal.jsx`. Mobile has **no shared bottom-sheet primitive at all** — at least a dozen files independently hand-roll the same backdrop-plus-sheet chrome: `AddToDiscoveryModal.jsx`, `BookmarkActionSheet.jsx`, `BookmarkComposer.jsx`, `AudienceSelector.jsx`, `LeftDrawer.jsx`, `UserMenu.jsx`, `ColorField.jsx`, `compose.js`, `admin/invites.js`, `admin/users.js`, `user/[id]/index.js`, `ReactButton.jsx`'s long-press picker (found during that component's own audit, see `components/ReactButton.md`), plus `BottomSheetPicker.jsx` (which is a real, legitimately-scoped component of its own — an option picker — but duplicates the *chrome* rather than building on a shared base). This list is very likely incomplete — it's from targeted greps, not an exhaustive sweep; treat "at least a dozen" as a floor, not a final count. Same failure mode as Field (6 web files) and Heading (75 web files, 300+ call sites), just the platforms swapped this time — confirms the note in `components/README.md`: check both platforms for "did anyone actually extract this," don't assume it's always web that's missing it.

**Contract:** build one shared `Sheet` primitive — backdrop (`Pressable`, ~40% black, tap-to-dismiss), an inner tap-swallowing `Pressable` so taps on the sheet itself don't dismiss it, a `SafeAreaView`-wrapped body pinned to the bottom, an optional title/eyebrow slot, a children slot, and an optional two-button footer action bar split evenly (the Cancel/Add pattern already in `AddToDiscoveryModal.jsx` — keep it, it's a good pattern, just make it part of the primitive instead of copied). `BottomSheetPicker` and every file listed above should build on `Sheet` rather than each maintaining its own copy of the backdrop/dismiss/safe-area logic.

## Smaller findings, not yet resolved

- **Web has an explicit header "Close" button; mobile relies only on backdrop-tap and the footer's Cancel button** — no equivalent explicit dismiss affordance in a sheet's header. Probably fine (bottom sheets conventionally work this way), but worth a deliberate check during implementation rather than assuming.
- **Mobile's sheets use `animationType="fade"`**, not `"slide"` — the sheet is positioned at the bottom via layout, but it fades into place rather than sliding up from off-screen, which is the more idiomatic bottom-sheet transition. Small polish item, not resolved here.
- **Web's `AddToCircleButton.jsx` doesn't use `Modal.jsx` at all** (found during that component's own audit — see `components/AddToCircle.md`) — it hand-rolls its own dialog with real behavior `Modal.jsx` lacks: ESC-to-close, body-scroll lock, and a more considered responsive layout (full-height on narrow viewports, centered `max-w-md` on wide ones vs. `Modal.jsx`'s fixed `max-w-lg`). Those three behaviors should probably become part of `Modal`'s own contract rather than staying a one-off improvement — not resolved here, flagging for whoever next touches this file.
- **Mobile's `ProfileActions.jsx` overflow (Block/Mute) menu is a third overlay shape** — anchored near its trigger via measured position, not a centered `Modal` or a bottom `Sheet`. Possibly worth a named third primitive (`Menu`?) if web has an equivalent anchored-dropdown pattern elsewhere (untraced so far) — not resolved, just noted so it isn't lost.

## Tokens used

- Palette: `base-100` (sheet/dialog body), `base-200`/`base-300` (dividers, header/footer borders), `primary`/`primary-content` (confirm action)
- Typography: `font-ui` throughout (chrome), `font-display` for web's dialog title (per the resolved Heading scale — likely `title` level)
