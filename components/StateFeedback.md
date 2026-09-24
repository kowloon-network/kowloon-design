# Spinner / EmptyState / ErrorState

**Purpose:** The three states every list/feed screen needs — loading, nothing to show, something went wrong.

## The largest raw duplication count found in this whole series

Web has three small, clean shared components (`Spinner.jsx`, `EmptyState.jsx`, `ErrorState.jsx`). Mobile has **none of the three** — every screen hand-rolls its own. Grepped counts: `ActivityIndicator` (loading) appears raw in **62 files**; empty-state text patterns in **24 files**; error/retry patterns in **38 files**. Bigger than Modal's dozen, on the same order as Heading's 75-file spread. Same failure mode this series keeps finding on mobile specifically (after Modal/Sheet) — confirms it's not a one-off.

## Real style drift, not just missing extraction

Sampled across admin screens, tab screens, and feed/circle/group/server screens — mobile's treatment is consistent *with itself* in the broad strokes but genuinely different from web's in register:

| | Web (`EmptyState`/`ErrorState`) | Mobile (sampled across ~10 files) |
|---|---|---|
| Empty-state text | `text-sm uppercase tracking-widest text-base-content/65` — reads as a small chrome label | `text-base` or `text-lg`, sentence case, `/60`–`/70` opacity — reads as a plain sentence, friendlier and larger |
| Error message | `text-sm uppercase tracking-widest text-error`, plus a `border-l-4 border-error` accent | `text-base text-error`, sentence case, no border accent at all |
| Retry action | "Try again" (underlined text link) | "Retry" (uppercase button, chrome-styled — inconsistently, since the message above it isn't) |

## OPEN — needs Josh's call: chrome-label register or conversational register?

This isn't a token-value drift, it's a real question about how the app *talks* to someone when there's nothing to show or something broke. Web's uppercase-label treatment matches the established chrome convention (Eyebrow, buttons, tags are all uppercase) — consistent with the rest of the system. Mobile's plain-sentence treatment is arguably better UX writing in the moment — "No circles yet. Create one first." reads as a person talking to you, not a system status label — even though it breaks from the uppercase convention. Not deciding this here. Worth noting mobile's retry button is already inconsistent with itself (uppercase button under a sentence-case message), so at minimum that internal mismatch should resolve one way or the other regardless of which register wins overall.

## Smaller, easy-to-resolve alongside the register question

- Copy: "Try again" (web) vs. "Retry" (mobile) — pick one wording once the register question is settled.
- Error accent: web's `border-l-4 border-error` treatment is a nice, legible "something's wrong" signal absent from every sampled mobile error state — worth adopting regardless of which text register wins, it's a structural affordance, not a register choice.

## Contract shape (once the register question is resolved)

- `Spinner`: size variants (`sm`/`md`/`lg`), optional centered/full-width wrapper mode. Lowest-risk of the three — RN's native `ActivityIndicator` doesn't offer much surface for visual drift beyond size/color, so this is mostly "wrap it in one reusable component with consistent size tokens" rather than a real design decision.
- `EmptyState`: message + optional action slot (matches web's `action` prop — an optional child element, e.g. a "Create one" button).
- `ErrorState`: message + optional retry callback, `role="alert"`/accessibility-announced on web (mobile should get an equivalent screen-reader announcement, not just visual styling).

## Tokens used

- Palette: `error` (message + accent), `primary` (spinner), `base-content` at reduced opacity (empty-state message)
- Typography: `font-ui` — register (uppercase-chrome vs. sentence-conversational) is exactly the open question above
