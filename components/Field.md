# Field

**Purpose:** A labeled text input — the base building block for every form on both platforms (login, register, profile, compose, admin settings).

## This audit found something bigger than Button's drift

Button existed as one real shared component per platform that had drifted apart. Field is worse: **web has no shared Field component at all.** Six files each independently define their own local `Field`:

| File | Label treatment | Hint/error | Notes |
|---|---|---|---|
| `LoginPage.jsx` | `text-base-content/50`, hint inline next to label | hint only, `/30` opacity, inline | |
| `RegisterPage.jsx` | identical to LoginPage | identical to LoginPage | byte-for-byte copy of LoginPage's version |
| `ProfilePage.jsx` | `/50` opacity | hint below children, italic, `font-reading`, `/40` | different structure *and* different styling from Login/Register |
| `EditPostPage.jsx` | `/50` opacity | no hint support at all | simplest variant |
| `NewPostPage.jsx` | identical to EditPostPage | no hint support | |
| `AdminSettingsPage.jsx` | `/60` opacity, renamed `FieldRow` | `description` prop, bottom-border divider, `py-4` row spacing | built for a settings list, not a form — arguably a legitimately different component, not drift |

Mobile has one real, shared `kowloon-mobile/src/components/ui/Field.jsx`, used everywhere via the login screen and elsewhere. So the asymmetry itself is the finding: mobile got this right, web didn't.

## RESOLVED 2026-09-23: underline for both platforms

Josh's call, citing Material Design's underline text fields as working precedent on both desktop and touch: **both platforms converge on web's existing underline treatment.** No fill color at all — `bg-transparent`, a `base-300` bottom border by default, `primary` on focus, `error` on error.

This also retired the `field` (cream) palette token entirely — it existed only to fill inputs like mobile's Field, and once Field has no fill, `field` had no remaining purpose. It turned out to be used across 40+ mobile files, not just `Field.jsx` (admin forms, bookmark/circle/group composers, replies, the search bar, drawer, a couple of buttons). Per Josh: for now, every former `field` usage should just resolve to `base-100` — the same as the surrounding background, i.e. no visible distinction — rather than trying to individually decide right now which ones were really "an input" versus "a button" versus "a recessed row." That's a deliberate placeholder, not a final per-case decision; revisit each of those 40+ sites when they're actually touched during implementation, not in a batch right now.

## Bug found while auditing mobile's Field.jsx (fixed by this resolution)

The component's own comment said: *"2px bottom [border] that shifts to primary on focus is future work — for now we render a 2px box on all sides for clarity."* The actual `className` had **no border classes at all** — it rendered with zero visible outline, relying only on the cream `bg-field` fill differing from whatever was behind it, with no focus-state change and an `error` prop that only reddened the message text, not the input. The underline resolution above fixes all three by construction — there's no separate "add a border" bug left once the input is underline-only with real border-color states for default/focus/error.

## States

| State | Contract |
|---|---|
| default | `border-b-2 border-base-300`, transparent fill |
| focus | border color to `primary` |
| error | border color to `error`, plus message text below (web today has no per-field error state at all, only a form-wide banner — this adds one; mobile today only reddens the message, not the input — this fixes that) |
| disabled | not audited on either platform yet — needs checking once implementation starts |

## Props (proposed contract, reconciling the six web variants down to one)

| Prop | Contract |
|---|---|
| `label` | Uppercase, letter-spaced, `font-ui` — matches the reconciled Button label treatment (§ chrome typography, `0.16em` tracking per Button.md, worth reusing here rather than each Field variant inventing its own tracking value) |
| `hint` | Small helper text below the input — every web variant that has this puts it in a different place/style; mobile's placement (below, not inline next to label) is the one to standardize on since it doesn't compete with the label for horizontal space |
| `error` | Replaces hint when present; reddens the input's own border/underline in addition to the message text |
| `value` / `onChangeText` (mobile) / `onChange` (web) | platform-native, not drift |
| `secureTextEntry` (mobile) / password-toggle via `PasswordInput` wrapper (web) | Both platforms already have a working show/hide password pattern — keep as-is, just make sure the underlying Field they wrap is the same one |
| `placeholder` | ✓ both — placeholder text showing a real example format (like `@you@example.com`) must never be forced uppercase, per IDEOLOGY.md §2 rule 4 |

`AdminSettingsPage`'s `FieldRow` (divider, row spacing, `description` instead of `hint`) is likely a **legitimately separate component** — a settings-list row, not a form field — and probably shouldn't be collapsed into Field. Worth its own short contract later rather than forcing it into this one.

## Tokens used

- Palette: `base-300` (default underline), `primary` (focus), `error`, `base-content` — no fill token; `field` is retired
- Typography: `font-ui`, label tracking `0.16em` (matches Button's reconciled value) rather than the four different values currently scattered across the six web files (`tracking-widest` in most, no explicit value overridden in others)
