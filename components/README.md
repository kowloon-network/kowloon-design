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

Only [`Button.md`](Button.md) exists so far, written as an audit of what the two existing `Button` components actually do today (they've already drifted from each other — see that file). The rest of the component library hasn't been speced yet; that's the actual work of the redesign pass, not something to backfill wholesale here.
