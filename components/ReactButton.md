# ReactButton

**Purpose:** The emoji-reaction control on posts and replies — one reaction per user, tap to add/change, tap-your-own to remove.

## Core logic already matches — this is a Field/TypeFilter-shaped audit, not a Card-shaped one

The one-reaction-per-user model (tap current emoji to clear, tap a different one to replace in one step, tap any with none set to add), the optimistic-update-with-rollback pattern, and the module-level emoji-list caching are implemented near-identically on both platforms, comments and all. The real findings are specific, fixable bugs, not a structural rebuild.

## RESOLVED: three real bugs, not judgment calls

1. **Web's picker icon uses FontAwesome** (`faFaceSmile`) — a second confirmed site for the icon migration already decided in IDEOLOGY.md §6 (lucide-react everywhere on web). Mobile already does this correctly (`Smile` from `lucide-react-native`).
2. **Drop shadow on both platforms, different mechanisms** — web's popup has `shadow-lg`; mobile's inline picker has `elevation: 6` (Android's native shadow system, triggered via React Native's `elevation` style prop). Both violate IDEOLOGY.md §2 rule 2 (no drop shadows, no exceptions). Fix: remove both; use a `border` (matching the picker's existing `border-2 border-primary` on web, which should stay) for separation instead.
3. **Web shows the reaction count twice per card — a real duplicate, not a style choice.** `PostReacts.jsx` already shows the total count flush-right above the action bar (`{total > 0 && <span>{total}</span>}`), and `ReactButton.jsx` *also* shows `{count > 0 && <span>{count}</span>}` next to the button itself inside `PostToolbar`. Mobile's `ReactButton.jsx` has an explicit comment documenting the fix already made there: *"Count lives in the react-summary row (feed) / ReactsBar (detail), not on the button (#79). The button just shows the react icon."* Web never got that fix. **Fix: remove the inline count from web's `ReactButton`**, matching mobile — the count already lives in `PostReacts`/`ReactCounts`.

## Platform note, not a gap: mobile's long-press bottom sheet

Mobile adds a second interaction web doesn't have: long-press opens a full bottom-sheet picker (4-per-row, emoji + name label) as "room to grow once the server's emoji set expands beyond six." Web's inline popup already wraps (`flex-wrap`) to fit more emojis without needing a secondary picker, which desktop has the screen space for and a phone doesn't — a legitimate platform difference, not something web needs to port. Worth noting for the Modal/Sheet migration, though: this bottom sheet is another site hand-rolling the same backdrop chrome `components/Modal.md` already flagged — one more addition to that migration list, not a new finding of its own.

## Tokens used

- Palette: `primary` (active/reacted state, popup border), `base-100`/`base-200` (popup background, hover), `base-content` at various opacities (inactive icon, count text)
- Typography: `font-ui` for count/name labels (chrome)
