# Heading / Eyebrow

**Purpose:** The masthead pattern that's the clearest single expression of the whole ideology (IDEOLOGY.md §1) — a small uppercase eyebrow label plus a bold display heading. Used for page titles, section titles, card titles, and the literal masthead wordmark.

## The biggest gap found so far

Field had 6 web files independently hand-rolling their own version. This is worse — **web has zero shared Heading or Eyebrow component**, and the pattern is used constantly:

- **Headings:** 75 web files reference `font-display` headings directly, with at least 14 distinct size/tracking/leading combinations in real use (`text-2xl tracking-wide` ×19, `text-3xl tracking-wide` ×17, `text-5xl tracking-wide` ×12, `text-4xl leading-none tracking-wide` ×10, and ten more variations down to one-off combinations like `text-9xl` for the login masthead).
- **Eyebrows:** the `font-ui text-xs uppercase tracking-widest text-base-content/N` pattern alone appears **300+ times** across the web codebase, with **18 distinct opacity values** in active use — everything from `/30` to `/80`, with no apparent system (`/50` is most common at 109 occurrences, `/40` at 72, `/60` at 63, then a long tail of one-off values like `/35`, `/45`, `/55`, `/65`, `/75`).

Mobile, by contrast, has one real shared `kowloon-mobile/src/components/ui/Heading.jsx` exporting both `Heading` and `Eyebrow`, used at only 9 call sites total, nearly all with no className override beyond a margin adjustment. Mobile got this right the same way it got Field right; web never extracted either.

Given this is the second time in two components that web turns out to have *zero* shared component where mobile has one clean one, this looks like a real pattern in how the two codebases were built, not a coincidence — worth assuming it's the default case for every remaining component until proven otherwise, not the exception.

## Bug found in mobile's Heading.jsx (comment vs. code, not code vs. contract)

The component's own comment says *"Editorial display heading — **serif**, tight tracking"* — but the actual implementation renders `font-ui` (sans), not `font-reading` (serif). This isn't actually a bug against the *resolved* type system: per IDEOLOGY.md §5, all chrome — including headings — is Inter/sans now, so the **code is correct and the comment is stale**. Worth a one-line comment fix during implementation, not a behavior change.

## RESOLVED 2026-09-23: a named scale, not freeform sizes

Given 14+ raw size combinations don't represent 14 deliberate choices, they cluster naturally into three real use cases:

| Level | Maps to | Use for |
|---|---|---|
| `display` | `text-6xl`–`text-9xl`, `leading-none` | The masthead wordmark itself (login/register hero, server branding) — rare, large, contextual |
| `title` | `text-3xl`–`text-4xl`, `leading-none` | Page-level headings ("Sign in to your account", settings page titles) |
| `heading` | `text-xl`–`text-2xl` | Section and card-level titles — the most common case by far (19+17 of the sampled occurrences) |

Sizes within each level can still flex slightly by context (a `title` might be `text-3xl` on a narrow mobile viewport and `text-4xl` on wide desktop) — the point of the scale is naming the *three real roles*, not pinning one pixel value forever.

## RESOLVED 2026-09-23: Eyebrow becomes fully fixed, no size/opacity variants

Unlike Heading, an eyebrow is inherently a small utility label — it doesn't need a scale, it needs **one** treatment used consistently. Confirmed:

- `font-ui`, uppercase, `text-xs`, `tracking-[0.25em]` (mobile's existing value — wider than Button's reconciled `0.16em` label tracking, and that's a legitimate typographic convention, not drift: smaller uppercase text conventionally gets *more* tracking to stay legible, the same reason a book's running-head is more widely spaced than its chapter titles)
- `text-base-content/60` as the single canonical opacity — mobile's existing value, and close to web's own most common value (`/50`, 109 occurrences), so this is converging toward what already exists on both sides rather than inventing a new number
- Context-appropriate overrides (e.g. `text-base-100/80` for an eyebrow on a dark/colored surface like the login sidebar) are fine and expected — that's a real background-contrast need, not drift. The drift is same-context instances landing on 18 different values for no reason.

## Platform notes

- No props beyond `children` and an optional `className` escape hatch — this is intentionally a thin wrapper, not a configurable component, which is part of why the web-side proliferation is surprising rather than expected.

## Tokens used

- Typography: `font-ui` for both Heading and Eyebrow (chrome, per the resolved type system — headings are not reader-controlled content, they're platform chrome even when they sit above reader-controlled body text)
- Palette: `base-content` at full opacity (Heading), `base-content/60` (Eyebrow default)
