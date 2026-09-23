# Kowloon Design

The non-platform-specific design source of truth for Kowloon: color tokens, typography tokens, and component contracts shared by [kowloon-frontend](https://github.com/kowloon-network/kowloon-frontend) (web) and [kowloon-mobile](https://github.com/kowloon-network/kowloon-mobile) (React Native).

## Why this repo exists

Web and mobile render with completely different primitives — real DOM + Tailwind v4 + DaisyUI on web, React Native + NativeWind on mobile — so component *code* can't be shared between them. What can be shared is the *design*: the token values or scales, and a written contract per component (variants, states, props) that both platforms are built and reviewed against. This repo is that shared layer. See [`components/README.md`](components/README.md) for how the contracts work.

This repo does not contain any renderable UI code. If you're looking for actual components, they live in `kowloon-frontend/src/components` (JSX) and `kowloon-mobile/src/components` (React Native).

## Structure

```
tokens/
  palette.json      — color tokens (light + dark), moved here from @kowloon/client
  typography.md      — font tokens; also documents a known drift between platforms (see below)
components/
  README.md          — contract format + how to use it
  TEMPLATE.md         — blank contract to copy for a new component
  Button.md           — first worked example, including an audit of current web/mobile drift
```

## Consuming this repo

Like the rest of the Kowloon repos, this isn't published to npm — it's consumed as a sibling checkout via a `file:` dependency, same pattern as `@kowloon/client`:

```json
"@kowloon/design": "file:../design"
```

Clone it next to the other repos: `~/Projects/kowloon/design`.

**Not yet wired in.** As of this writing, `@kowloon/client` still ships `theme/palette.json` directly and the frontend/mobile still import from there. Moving them over to `@kowloon/design` is a real (if small) cross-repo change — see the open item below before doing that.

## Open items

- **Typography drift**: web's chrome fonts are Inter (display) / IBM Plex Sans (ui) / Source Serif 4 (reading); mobile's are Inter (ui) / Lora (reading), with no display font distinction. These were never reconciled. See `tokens/typography.md`.
- **Button drift**: web's `Button` has an `accent` variant and a `size` (sm/md/lg) prop that mobile's doesn't; mobile's has a `loading` state that web's doesn't. See `components/Button.md`.
- Rewiring `@kowloon/client`, `kowloon-frontend`, and `kowloon-mobile` to actually consume `tokens/palette.json` from here instead of the copy in `client` — not done yet, needs its own pass.
