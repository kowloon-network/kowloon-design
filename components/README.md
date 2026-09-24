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

[`Button.md`](Button.md), [`Field.md`](Field.md), [`Heading.md`](Heading.md), [`Card.md`](Card.md), [`Avatar.md`](Avatar.md), [`Reply.md`](Reply.md), [`Modal.md`](Modal.md), [`TypeFilter.md`](TypeFilter.md), [`ReactButton.md`](ReactButton.md), [`Toast.md`](Toast.md), and [`Timestamp.md`](Timestamp.md) are all fully resolved. The rest of the component library hasn't been speced yet; that's the actual work of the redesign pass, not something to backfill wholesale here.

**Timestamp found an inconsistency that's already live within web itself, not just web-vs-mobile.** Web's `Timestamp.jsx` has two different relative-time algorithms gated by a `compact` prop; only `PostMeta.jsx` (post cards) passes it. Every other usage — replies, notifications, pages — gets a different, cruder format. A reply shows "2h ago" directly under a post showing "2h." Resolved on mobile's single algorithm (already used unconditionally everywhere there), extracted to one shared place both platforms import rather than the two-files-plus-a-reminder-comment setup that was there before ("keep in sync with mobile's version" is not a sync mechanism).

**Lessons carried forward, still holding:** both platforms can independently disagree with this doc and be right (Avatar's circular-avatar exception); "no shared component" isn't the only failure mode — a shared component can exist as a monolith with an internal duplicate (Card), exist and still hide a real bug (ReactButton's duplicate count, per mobile's own comment referencing a fix web never got — issue #79), or not exist as a *feature* at all on one platform even though the underlying need obviously applies to both (Toast). Real behavioral bugs and feature gaps are turning out to be at least as common a finding as visual/structural drift — keep actively looking for both on every remaining audit.
