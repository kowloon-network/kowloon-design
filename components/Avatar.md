# Avatar

**Purpose:** The identity image for a user (circular), or a circle/group (hexagon-masked) — appears in bylines, member lists, selectors, and cards throughout the app.

## RESOLVED 2026-09-24: circular avatars are a deliberate, named exception to "no rounded corners"

Both platforms had already, independently, made user avatars circular before this doc existed — `kowloon-frontend/src/components/ui/UserAvatar.jsx` and `kowloon-mobile/src/components/posts/Avatar.jsx` carry nearly identical comments ("person = circle is universal convention" / "circular user avatar — person = circle is universal convention"). That directly contradicted IDEOLOGY.md §2 rule 1 as originally written, which listed avatars in the no-exceptions no-rounded-corners rule. Rule 1 is now amended: user avatars are circular by design, Circle/Group icons stay hexagon-masked (their own brand mark, not a rounded-corner exception), everything else stays square. See IDEOLOGY.md §2 for the amended rule and rationale.

## RESOLVED 2026-09-24: no drop shadow, on either platform

Web's `UserAvatar` has a real `boxShadow: '0 1px 2px rgba(0,0,0,0.18)'` — mobile's `Avatar.jsx` has none. This one wasn't a judgment call, it's a straightforward violation of IDEOLOGY.md §2 rule 2 (no drop shadows or elevation blur, no exceptions granted here). Fix: remove the shadow from web's `UserAvatar`.

## Structural finding: web hand-rolls the hex-mask pattern instead of having a shared component

Mobile has a clean three-layer structure: `HexAvatar` (the actual hexagon-clip primitive, SVG `ClipPath`-based) → `CircleAvatar`/`GroupAvatar` (thin wrappers supplying the right fallback icon and color). Web has no equivalent shared component — the hex-mask pattern (`mask-image: url(/hex-mask.svg)`) is hand-rolled as a local `hexMask` style object, independently redefined in at least two files (`CircleSelector.jsx`, `CirclesPage.jsx`), with the actual `<img style={hexMask}>` markup duplicated four times across those files for user, circle, and group icons at different sizes. Same pattern as Field and Heading: mobile extracted a shared component, web didn't. Web's `CircleIcon.jsx` is a separate, correctly-scoped thing — a generic type-glyph (not per-entity) used only as the fallback when a circle/group has no custom icon; it doesn't need to change.

**Contract:** both platforms get a `HexAvatar` primitive (hexagon-clip an image or a fallback fill+glyph) plus `CircleAvatar`/`GroupAvatar` wrappers, mirroring mobile's existing structure — web needs to build the shared components and replace its four duplicated inline usages, not invent a new structure.

## Variants

| Variant | Shape | Use for |
|---|---|---|
| `user` | Circle | Any person — byline avatars, member lists, profile |
| `circle` / `group` | Hexagon mask | Circle and Group icons, using the same hex geometry both already agree on |

## States

| State | Contract |
|---|---|
| has custom icon | Image, cover-fit, clipped to the variant's shape |
| no custom icon (user) | Filled color block + initial letter (web: `primary` fill; mobile: `secondary` fill — **minor inconsistency, not yet reconciled, pick one during implementation**) |
| no custom icon (circle/group) | Filled color block + `Users` glyph (mobile's existing fallback) rather than an initial letter — a circle/group doesn't have a single "name's first letter" the way a person does |
| image fails to load | Falls back to the no-custom-icon treatment above; both platforms already handle this correctly (`onError`/`imgError` state) |

## Props

| Prop | Contract |
|---|---|
| `size` | Numeric or `sm`/`md`/`lg` — both platforms already support this, just via different mechanisms (web: Tailwind size classes; mobile: numeric px passed straight through) — platform-native, not drift |
| `icon`/`uri` | The image URL |
| fallback content | Initial letter (user) or glyph (circle/group), per States above |

## Platform notes

- Web achieves the hexagon clip via CSS `mask-image: url(/hex-mask.svg)`; mobile via `react-native-svg`'s `ClipPath`/`Polygon`. Different mechanisms for the same visual result — expected, not drift.
- Web's `isCurrentUser` special case (prefer the live authenticated user's own profile icon over whatever was baked into a post at creation time) has no visible mobile equivalent in `Avatar.jsx` — worth checking whether mobile has this logic elsewhere or is missing it; not resolved in this pass.

## Tokens used

- Palette: `primary` or `secondary` (fallback fill — reconcile which, see States), `base-200` (mobile's image-loading background)
