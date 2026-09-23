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

## A real structural difference, not just a values question

This is a bigger decision than Button's accent-variant question — the two platforms use different **input metaphors**, not just different numbers:

- **Web:** underline only — `bg-transparent border-b-2 border-base-300 focus:border-primary`. Like a blank on a printed form.
- **Mobile:** filled box — `bg-field` (solid cream), no border at all currently (see bug below). A clearer touch target.

**OPEN — needs Josh's call:** converge on one metaphor for both platforms, or keep this as a documented, intentional platform difference (underline suits a wide desktop form; a filled box gives a bigger, clearer tap target on a small touch screen)? Recommend the latter — this feels more like "type is set differently in a magazine vs. a paperback" than actual drift, but it should be a decision either way, not an accident.

## Bug found while auditing mobile's Field.jsx

The component's own comment says: *"2px bottom [border] that shifts to primary on focus is future work — for now we render a 2px box on all sides for clarity."* The actual `className` has **no border classes at all** — `bg-field px-3 py-3 font-ui text-base text-base-content`. It currently renders with zero visible outline, relying only on the cream `bg-field` fill differing from whatever's behind it. There's also no focus-state style change, and the `error` prop only reddens the message text below, not the input itself. Treat all three as real bugs to fix during implementation, not as intentional restraint.

## States

| State | Web today | Mobile today | Contract |
|---|---|---|---|
| default | ✓ | ✓ (no visible border — bug above) | ✓ both, with mobile's border bug fixed |
| focus | ✓ (`focus:border-primary`) | ✗ (no focus style at all) | ✓ both |
| error | ✗ (no per-field state; only a form-wide banner exists) | partial (message text only, no input outline change) | ✓ both — input itself should show `error` color on its border/underline, not just the message below |
| disabled | not audited on either platform | not audited | needs checking once implementation starts |

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

- Palette: `base-300` (web underline default), `primary` (focus), `error`, `field` (mobile fill), `base-content`
- Typography: `font-ui`, label tracking should match Button's reconciled `0.16em` rather than the four different values currently scattered across the six web files (`tracking-widest` in most, no explicit value overridden in others)
