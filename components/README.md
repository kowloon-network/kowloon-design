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

[`Button.md`](Button.md), [`Field.md`](Field.md), [`Heading.md`](Heading.md), [`Card.md`](Card.md), [`Avatar.md`](Avatar.md), [`Reply.md`](Reply.md), [`Modal.md`](Modal.md), [`TypeFilter.md`](TypeFilter.md), and [`ReactButton.md`](ReactButton.md) are all fully resolved. The rest of the component library hasn't been speced yet; that's the actual work of the redesign pass, not something to backfill wholesale here.

**ReactButton found a real duplicate-count bug web never fixed.** Mobile's own code comment documents a resolved decision (issue #79): the reaction count moved out of the button itself into the react-summary row elsewhere on the card. Web never got that fix — `PostReacts` and `ReactButton` both show the same count, doubled up on every card. Also found: web's picker icon is still FontAwesome (second confirmed site needing the §6 lucide migration), and both platforms have a real drop shadow on their picker popups (web: `shadow-lg`; mobile: `elevation`), both removed.

**Lessons carried forward, still holding:** both platforms can independently disagree with this doc and be right (Avatar's circular-avatar exception); "no shared component" isn't the only failure mode — a shared component can exist as a monolith with an internal duplicate (Card), or exist and still have real bugs nobody caught because nothing was comparing the two platforms directly (Card's byline-typography bug, now ReactButton's duplicate count). That last one is turning out to be the single most common category of finding across this whole series — worth actively looking for on every remaining audit, not just structural drift.
