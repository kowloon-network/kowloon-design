# Kowloon Design

The non-platform-specific design source of truth for Kowloon: color tokens, typography tokens, and component contracts shared by [kowloon-frontend](https://github.com/kowloon-network/kowloon-frontend) (web) and [kowloon-mobile](https://github.com/kowloon-network/kowloon-mobile) (React Native).

**Start with [`IDEOLOGY.md`](IDEOLOGY.md)** — the design principles and rules everything else in this repo (and every component contract) should trace back to. Read that before adding a new token or contract.

## Why this repo exists

Web and mobile render with completely different primitives — real DOM + Tailwind v4 + DaisyUI on web, React Native + NativeWind on mobile — so component *code* can't be shared between them. What can be shared is the *design*: the token values or scales, and a written contract per component (variants, states, props) that both platforms are built and reviewed against. This repo is that shared layer. See [`components/README.md`](components/README.md) for how the contracts work.

This repo does not contain any renderable UI code. If you're looking for actual components, they live in `kowloon-frontend/src/components` (JSX) and `kowloon-mobile/src/components` (React Native).

## Structure

```
IDEOLOGY.md          — design principles and rules; read this first
tokens/
  palette.json      — color tokens (light + dark), moved here from @kowloon/client
  typography.md      — chrome vs. reader-controlled content typography, fully resolved
components/
  README.md          — contract format + how to use it
  TEMPLATE.md         — blank contract to copy for a new component
  Button.md           — first worked example, fully resolved contract
reference/
  style-guide.html   — generated snapshot: the full palette + Button contract rendered live, self-contained (open directly in a browser, no build step)
```

## Consuming this repo

Like the rest of the Kowloon repos, this isn't published to npm — it's consumed as a sibling checkout via a `file:` dependency, same pattern as `@kowloon/client`:

```json
"@kowloon/design": "file:../design"
```

Clone it next to the other repos: `~/Projects/kowloon/design`.

**Not yet wired in.** As of this writing, `@kowloon/client` still ships `theme/palette.json` directly and the frontend/mobile still import from there. Moving them over to `@kowloon/design` is a real (if small) cross-repo change — see the open item below before doing that.

## Open items

Every decision opened in `IDEOLOGY.md` §10 is resolved (palette, chrome/content typography, icons, Button contract) — see `reference/style-guide.html` for the current state rendered live.

- Rewiring `@kowloon/client`, `kowloon-frontend`, and `kowloon-mobile` to actually consume this repo instead of the copy in `client` — **on hold until the component library is fully specced**, per Josh's instruction.
- The component library itself: only Button is specced so far. Everything else in `kowloon-frontend/src/components` and `kowloon-mobile/src/components` still needs a contract.
