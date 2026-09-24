# Component contracts

A contract is a short spec for one component: what variants and states it has, what props drive them, and what's platform-specific. It's the thing both the JSX implementation and the React Native implementation are built against and reviewed for drift against — instead of one platform's code being copied into the other after the fact (which is how the current codebase ended up with, e.g., three separately-hand-maintained copies of the post composer on web alone).

## Format

Copy [`TEMPLATE.md`](TEMPLATE.md) for a new component. A contract has:

- **Purpose** — one line, what it's for
- **Variants** — the meaningfully different visual treatments (e.g. primary/secondary/ghost)
- **States** — orthogonal to variants (disabled, loading, focused, error...)
- **Props** — the contract's actual API surface; both platforms should expose the same props unless there's a stated reason not to
- **Platform notes** — anything that has to differ and why (e.g. `onPress` vs `onClick`, hover states that don't exist on touch)
- **Tokens used** — which palette/typography tokens this component consumes, so a token change can be traced to what it affects

## How to use this when redesigning a component

1. Write or update the contract first — decide the variants/states/props before touching code.
2. Implement (or update) both the web and mobile version against the same contract.
3. If a platform needs to diverge from the contract, that divergence goes in "Platform notes" — it should be a documented decision, not silent drift.

## Current state

[`Button.md`](Button.md), [`Field.md`](Field.md), [`Heading.md`](Heading.md), [`Card.md`](Card.md), and [`Avatar.md`](Avatar.md) are all fully resolved. [`Reply.md`](Reply.md) is mostly resolved, one open question (see below). The rest of the component library hasn't been speced yet; that's the actual work of the redesign pass, not something to backfill wholesale here.

**Reply's audit found the two platforms already agree more than they disagree** — a first for this series. Both have one real `Reply` component with genuinely similar 2-level nesting/indent logic already. The real findings were a gap (reply bodies aren't wired to reader-controlled typography on either platform, despite that being explicitly in scope per the resolved typography boundary) and dead code (`ReplyList.jsx` on web is unused anywhere and would be broken if it were — it doesn't match the shape `buildReplyTree` actually produces). One open item: whether the hairline rule between reply rows should be removed the same way it just was on `PostCard`/`EventCard`, or whether a threaded conversation is different enough from a feed of independent posts to keep it.

**Avatar's audit found a real conflict with IDEOLOGY.md itself, not just drift** — both platforms had already, independently, made user avatars circular, contradicting the no-rounded-corners rule as originally written (which listed avatars with no exception). Confirmed as a deliberate, permanent exception rather than a bug: IDEOLOGY.md §2 rule 1 is now amended. Worth remembering for future audits — when both platforms agree with each other and disagree with this doc, that's real signal the doc missed something, not necessarily two platforms drifting the same wrong way.

**The "web has zero shared component" pattern from Field and Heading isn't universal** — Card broke it. Both platforms have a real `PostCard`, just structured very differently: web decomposes into five components (`PostMeta`/`PostBody`/`PostReacts`/`PostToolbar`/`EventCard`), mobile is one 413-line file handling every post type inline, plus a second unshared copy of the same rendering logic in mobile's own separate `PostBody.jsx` (used only by the detail screen). Worth checking both failure modes — "no shared component" and "shared component that's ballooned into a monolith with an internal duplicate" — on every future audit, not just the first one.

Card's audit also turned up a real cross-platform typography bug (mobile applying the reader's chosen font to post bylines, which the resolved typography boundary explicitly says shouldn't happen) and confirmed two genuinely new features to build, not just reconcile: a capped 2×2 media grid with a "see more" link in the feed card, and porting web's calendar-block Event treatment to mobile, which has none today.
