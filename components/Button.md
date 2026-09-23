# Button

**Purpose:** The primary tappable/clickable control — solid-fill button with an uppercase, letter-spaced label, sharp corners (no rounding), theme-token colors.

This spec is written as an **audit of the two components as they exist today** (`kowloon-frontend/src/components/ui/Button.jsx`, `kowloon-mobile/src/components/ui/Button.jsx`) — they've already drifted from a common contract without anyone deciding that. Treat the "Should be" column as the thing to reconcile toward, not as already true.

## Variants

| Variant | Web | Mobile | Should be |
|---|---|---|---|
| `primary` | ✓ | ✓ | ✓ |
| `secondary` | ✓ | ✓ | ✓ |
| `accent` | ✓ | ✗ | Decide: does mobile need accent, or should web drop it? |
| `ghost` | ✓ | ✓ | ✓ |

## States

| State | Web | Mobile | Should be |
|---|---|---|---|
| `disabled` | ✓ (`disabled` prop, `opacity-40`) | ✓ (`disabled` prop, `/60` opacity on fill) | ✓, but reconcile the actual disabled treatment (different opacity values) |
| `loading` | ✗ | ✓ (spinner replaces label, implicitly disables) | Probably both — web forms already need a loading-submit state, currently hand-rolled per call site instead of in Button |

## Props

| Prop | Web | Mobile | Notes |
|---|---|---|---|
| variant | `primary\|secondary\|accent\|ghost` | `primary\|secondary\|ghost` | see Variants above |
| size | `sm\|md\|lg` | *(none — fixed padding)* | mobile has no size scale at all |
| disabled | ✓ | ✓ | |
| loading | ✗ | ✓ | |
| label / children | `children` (JSX) | `label` (string) | genuine platform difference, not drift — RN can't take arbitrary children the same way, this is fine |
| onClick / onPress | `onClick` | `onPress` | platform-native event name, fine as-is |
| type | `type` (`button\|submit`) | *(n/a — RN has no form submit)* | fine as-is |

## Platform notes

- `label`/`children` and `onClick`/`onPress` naming differences are legitimate platform conventions, not drift to fix.
- Mobile's `android_ripple` touch feedback has no web equivalent (web gets `hover:opacity-90` instead) — expected, not a gap.

## Tokens used

- Palette: `primary`/`primary-content`, `secondary`/`secondary-content`, `accent`/`accent-content` (web only currently), `base-content`, `base-200`
- Typography: `font-ui`, uppercase, `tracking-widest` (web) / `tracking-[0.18em]` (mobile) — **also drifted**, should be one shared letter-spacing value
