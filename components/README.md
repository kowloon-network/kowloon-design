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

[`Button.md`](Button.md), [`Field.md`](Field.md), [`Heading.md`](Heading.md), [`Card.md`](Card.md), [`Avatar.md`](Avatar.md), [`Reply.md`](Reply.md), [`Modal.md`](Modal.md), and [`TypeFilter.md`](TypeFilter.md) are all fully resolved. The rest of the component library hasn't been speced yet; that's the actual work of the redesign pass, not something to backfill wholesale here.

**TypeFilter is the cleanest audit in the series** — web's own code comment says it was already "converged with the mobile app's TypeFilter," and the solo-first-tap logic is implemented identically on both platforms. Only real difference: inactive icons went grayscale on web vs. a faded tint of the type's own color on mobile. Resolved on mobile's approach — grayscale erases the semantic color-coding signal §4 depends on, even in the faded state. Good reminder that not every audit surfaces a big structural problem; sometimes the answer really is "mostly fine, one small thing."

**Modal/Sheet flipped the usual pattern — this time it's mobile duplicating, not web.** Web has one clean `Modal.jsx` (centered dialog); mobile has no shared bottom-sheet primitive at all, with at least a dozen files hand-rolling the same backdrop chrome independently. The shape difference itself (centered dialog vs. bottom sheet) is a legitimate platform convention, kept as two differently-named primitives (`Modal` / `Sheet`) rather than forced into one — but mobile needs the same "extract a shared component" fix Field and Heading needed on web. Confirms the standing rule: check *both* platforms for missing extraction, never assume it's always the same one.

**Lessons carried forward from earlier audits, still holding:**
- **Both platforms can independently disagree with this doc and be right** — Avatar's circular-user-avatar exception was confirmed by both platforms agreeing with each other before this doc existed; IDEOLOGY.md §2 rule 1 was amended rather than enforced against two consistent, reasoned decisions.
- **"No shared component" isn't the only failure mode** — Card's `PostCard` exists on both platforms, just shaped very differently (web: five decomposed components; mobile: one 413-line monolith with an internal *second*, unshared duplicate in its own `PostBody.jsx`).
- **A shared component existing doesn't mean it's bug-free** — Card also surfaced a real cross-platform typography violation (mobile applying reader-controlled font to post bylines) and two genuinely new features to build, not reconcile (capped 2×2 media grid, ported Event calendar-block).
