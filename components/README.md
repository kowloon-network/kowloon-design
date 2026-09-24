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

[`Button.md`](Button.md), [`Field.md`](Field.md), [`Heading.md`](Heading.md), [`Card.md`](Card.md), [`Avatar.md`](Avatar.md), [`Reply.md`](Reply.md), [`Modal.md`](Modal.md), [`TypeFilter.md`](TypeFilter.md), [`ReactButton.md`](ReactButton.md), [`Toast.md`](Toast.md), [`Timestamp.md`](Timestamp.md), [`AddToCircle.md`](AddToCircle.md), [`Notification.md`](Notification.md), and [`StateFeedback.md`](StateFeedback.md) are all fully resolved. The rest of the component library hasn't been speced yet; that's the actual work of the redesign pass, not something to backfill wholesale here.

**StateFeedback found the largest raw duplication count in the series** — web has three clean shared components (Spinner/EmptyState/ErrorState); mobile has none of the three, hand-rolled across 62 files for loading, 24 for empty states, 38 for errors. Bigger than Modal's dozen. Real register drift too, not just missing extraction: web's empty/error text read as a small uppercase chrome label, mobile's read as a plain conversational sentence. Resolved on web's chrome-label register (consistent with Eyebrow/buttons/tags), with mobile's shorter "Retry" wording kept over web's "Try again," plus web's `border-l-4` error accent bar adopted on both.

**Notification split one overloaded signal into two.** Web used a dot for unread and colored icons per type; mobile used a left-edge bar for unread and left type uncolored. Resolved: the dot is unread (both platforms), a left-edge color bar is type (both platforms, echoing `Toast.md`'s same pattern) — two independent signals instead of one shape trying to carry both meanings. Also: removed web's still-active `follow` notification type (icon/color/routing) to match mobile's explicit, policy-cited omission — the third audit in a row to find a web component file that looked live but wasn't (`ReplyList.jsx`, `FollowButton.jsx`'s chain, now `NotificationItem.jsx`); always verify a file is actually imported by the real page before trusting it.

**Lessons carried forward, still holding:** both platforms can independently disagree with this doc and be right (Avatar's circular-avatar exception); a shared component can exist as a monolith with an internal duplicate (Card), hide a real bug nobody caught by comparing platforms (ReactButton, Timestamp), or not exist as a *feature* at all on one platform despite the need obviously applying to both (Toast). Dead code that looks live is now a confirmed recurring pattern on web specifically (three instances) — check whether a file is actually imported before trusting it, every time.
