# TypeFilter

**Purpose:** The icon-row filter for post type (Note/Article/Media/Link/Event) atop a feed. No "All" button — an empty selection already means "all types."

## The cleanest audit in this series — logic is already converged

Web's `TypeFilter.jsx` opens with its own comment: *"shared post-type filter row (icon-only), converged with the mobile app's TypeFilter."* The solo-on-first-tap behavior (documented standing rule: first tap from "all" solos that type; tapping an active type removes it and an empty set wraps back to "all"; tapping an inactive type adds it and a full set normalizes back to `[]`) is implemented identically on both platforms, variable names and all. Someone already did the reconciliation work here — first time in this series a component didn't need it.

## RESOLVED 2026-09-24: inactive icons keep their type color, faded — not grayscale

The one real difference: mobile fades an inactive icon to `ink(0.15)` — 15% opacity of its own type color, so an inactive Article icon still reads as a very faint teal. Web strips color entirely via `opacity-25 grayscale` — an inactive icon reads as faint gray regardless of type. Resolving on mobile's approach: grayscale fully erases the semantic color-coding signal (IDEOLOGY.md §4 — content-type color is meant to be meaningful wayfinding) even in the faded state, where a tinted-but-faint icon preserves a whisper of "this is still nameable as its type" while clearly reading as inactive. Web drops `grayscale`, both platforms fade to a low opacity of the icon's own color (exact percentage — mobile's 15% or a nearby value — to be tuned during implementation, not pinned here).

## Tokens used

- Palette: `post-note`/`post-article`/`post-media`/`post-link`/`post-event` (active state, full color), same tokens at reduced opacity (inactive state)
