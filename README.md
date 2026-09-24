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
  Button.md           — fully resolved contract
  Field.md             — fully resolved contract (underline for both platforms)
  Heading.md           — fully resolved contract (named scale + fixed Eyebrow)
  Card.md              — fully resolved contract (PostCard/EventCard)
  Avatar.md             — fully resolved contract (circular for people, hex for Circle/Group)
  Reply.md              — fully resolved contract (hairline rule kept, deliberately unlike Card)
  Modal.md              — fully resolved contract (Modal on web, Sheet on mobile — different shapes, kept)
  TypeFilter.md          — fully resolved contract (grayscale dropped, fade the type's own color instead)
  ReactButton.md          — fully resolved contract (fixes a real duplicate-count bug on web)
  Toast.md                — fully resolved contract (real feature gap: mobile has no toast system)
  Timestamp.md             — fully resolved contract (one algorithm, not three; web's own replies/posts disagreed)
  AddToCircle.md            — fully resolved contract (dead FollowButton code found and flagged for deletion)
  Notification.md            — fully resolved contract (dot for unread, left color bar for type)
  StateFeedback.md           — fully resolved contract (web's chrome-label register, "Retry" wording, error accent bar)
reference/
  style-guide.html   — generated snapshot: the full palette + Button + Field + Heading/Eyebrow, rendered live, self-contained (open directly in a browser, no build step) — the rest not yet added, either too composite/data-driven for the token-swatch format or just not gotten to yet
```

## Consuming this repo

Like the rest of the Kowloon repos, this isn't published to npm — it's consumed as a sibling checkout via a `file:` dependency, same pattern as `@kowloon/client`:

```json
"@kowloon/design": "file:../design"
```

Clone it next to the other repos: `~/Projects/kowloon/design`.

**Not yet wired in.** As of this writing, `@kowloon/client` still ships `theme/palette.json` directly and the frontend/mobile still import from there. Moving them over to `@kowloon/design` is a real (if small) cross-repo change — see the open item below before doing that.

## Open items

Every decision opened in `IDEOLOGY.md` §10 is resolved, and so are Button, Field, Heading/Eyebrow, Card/EventCard, Avatar, Reply, Modal/Sheet, TypeFilter, ReactButton, Toast, Timestamp, AddToCircle, Notification, and StateFeedback. `reference/style-guide.html` covers Button/Field/Heading; the rest aren't added yet.

- Rewiring `@kowloon/client`, `kowloon-frontend`, and `kowloon-mobile` to actually consume this repo instead of the copy in `client` — **on hold until the component library is fully specced**, per Josh's instruction. Real implementation work is piling up behind that hold — see each component's own `.md` for its specific to-do list (token migrations, new features to build, dead code to delete, bugs to fix). It's substantial at this point; expect a real implementation project once the spec pass is done, not a quick find-and-replace. Toast and StateFeedback are the two largest single implementation items so far (a whole notification system on mobile; 62+24+38 files to migrate onto three new shared components).
- The component library itself: Button, Field, Heading, Card, Avatar, Reply, Modal, TypeFilter, ReactButton, Toast, Timestamp, AddToCircle, Notification, and StateFeedback are specced. Everything else in `kowloon-frontend/src/components` and `kowloon-mobile/src/components` still needs a contract — per `components/README.md`, check for *both* known failure modes (either platform can be the one with no shared component, or the one whose shared component has ballooned into an unmaintainable monolith), and actively look for real behavioral bugs/feature gaps/dead code, not just visual/structural drift.
- Also filed: [issue #69](https://github.com/kowloon-network/kowloon/issues/69), large media uploads need progress + background handling — a real gap found while discussing Toast, not part of this repo's scope but worth remembering it's out there.
- Also filed: [issue #69](https://github.com/kowloon-network/kowloon/issues/69), large media uploads need progress + background handling — a real gap found while discussing Toast, not part of this repo's scope but worth remembering it's out there.
