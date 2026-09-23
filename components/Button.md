# Button

**Purpose:** The primary tappable/clickable control — solid-fill button with an uppercase, letter-spaced label, sharp corners (no rounding), theme-token colors.

Originally written as an audit of the two components as they exist today (`kowloon-frontend/src/components/ui/Button.jsx`, `kowloon-mobile/src/components/ui/Button.jsx`), which had already drifted from a common contract without anyone deciding that. **RESOLVED 2026-09-23** — the columns below are now the target contract, not just an audit. Not yet implemented in either repo (this repo doesn't wire into other repos' code yet); tracked as implementation work for the component pass.

## Variants

| Variant | Web today | Mobile today | Contract |
|---|---|---|---|
| `primary` | ✓ | ✓ | ✓ |
| `secondary` | ✓ | ✓ | ✓ |
| `accent` | ✓ | ✗ | ✓ both — add to mobile now that `accent` has a resolved value (`#e75423`/`#e8987d`, IDEOLOGY.md §4) |
| `ghost` | ✓ | ✓ | ✓ |

## States

| State | Web today | Mobile today | Contract |
|---|---|---|---|
| `disabled` | ✓ (dims whole button to 40% opacity) | ✓ (dims only the fill to 60%, label/spinner stay full opacity) | Mobile's mechanism, both platforms — dimming the label along with the fill (web's current approach) makes it harder to read *why* a control is disabled, which is the one thing legibility matters most for in that state |
| `loading` | ✗ | ✓ (spinner replaces label, implicitly disables) | ✓ both — web forms already need this; today it's hand-rolled per call site instead of living in Button |

## Props

| Prop | Web today | Mobile today | Contract |
|---|---|---|---|
| variant | `primary\|secondary\|accent\|ghost` | `primary\|secondary\|ghost` | `primary\|secondary\|accent\|ghost`, both |
| size | `sm\|md\|lg` | *(none — fixed padding)* | `sm\|md\|lg`, both — mobile adopts web's exact classes (`px-3 py-1.5 text-xs` / `px-4 py-2 text-sm` / `px-6 py-3 text-base`); NativeWind supports the same utility strings, no separate scale to invent |
| disabled | ✓ | ✓ | ✓ both |
| loading | ✗ | ✓ | ✓ both |
| label / children | `children` (JSX) | `label` (string) | unchanged — genuine platform difference, not drift. RN can't take arbitrary children the same way |
| onClick / onPress | `onClick` | `onPress` | unchanged — platform-native event name |
| type | `type` (`button\|submit`) | *(n/a)* | unchanged — RN has no form submit |

## Platform notes

- `label`/`children` and `onClick`/`onPress` naming differences are legitimate platform conventions, not drift to fix.
- Mobile's `android_ripple` touch feedback has no web equivalent (web gets `hover:opacity-90` instead) — expected, not a gap.

## Tokens used

- Palette: `primary`/`primary-content`, `secondary`/`secondary-content`, `accent`/`accent-content`, `base-content`, `base-200`
- Typography: `font-ui` (Inter, per the resolved chrome type system), uppercase, **letter-spacing reconciled to `0.16em`** on both platforms — web's `tracking-widest` (0.1em) and mobile's `tracking-[0.18em]` were two different arbitrary values; this splits between them rather than picking either platform's number by default.
