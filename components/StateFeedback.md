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

## RESOLVED 2026-09-24: web's register, "Retry" for the wording

Josh's call: web's chrome-label register wins — `text-sm uppercase tracking-widest`, consistent with Eyebrow/buttons/tags across the rest of the system. Mobile's empty/error text moves off its current plain-sentence treatment to match. Wording is a mix of both platforms' instincts, not a pure "web wins" — the retry action reads **"Retry"** (mobile's shorter wording), not web's "Try again." That also resolves mobile's internal inconsistency (an uppercase Retry button under a sentence-case message) automatically, since the message above it is uppercase now too.

## Also adopted: web's error accent bar

`border-l-4 border-error` — a structural affordance, not a register choice, and it was missing from every sampled mobile error state. Both platforms get it.

## Contract shape

- `Spinner`: size variants (`sm`/`md`/`lg`), optional centered/full-width wrapper mode. Lowest-risk of the three — RN's native `ActivityIndicator` doesn't offer much surface for visual drift beyond size/color, so this is mostly "wrap it in one reusable component with consistent size tokens" rather than a real design decision.
- `EmptyState`: message + optional action slot (matches web's `action` prop — an optional child element, e.g. a "Create one" button).
- `ErrorState`: message + optional retry callback, `role="alert"`/accessibility-announced on web (mobile should get an equivalent screen-reader announcement, not just visual styling).

## Tokens used

- Palette: `error` (message + accent bar), `primary` (spinner), `base-content` at reduced opacity (empty-state message)
- Typography: `font-ui`, uppercase, `tracking-widest` — chrome-label register, both platforms
