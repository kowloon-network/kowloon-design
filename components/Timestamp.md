# Timestamp

**Purpose:** Relative time display ("4m," "3h," "2d") on posts, replies, notifications, pages.

## RESOLVED 2026-09-24: one algorithm, everywhere — not three

Web's `Timestamp.jsx` actually contains **two different relative-time algorithms**: `formatCompact` (used only when a `compact` prop is passed) and `formatRelative` (the default when it isn't) — different phrasing ("4m" vs "4m ago"), different thresholds (`formatRelative` has no week bucket and falls back to an absolute date after 30 days; `formatCompact` has a week bucket and holds off on the absolute date until 5 weeks). Checking every call site: **only `PostMeta.jsx` passes `compact`.** `Reply.jsx`, `NotificationItem.jsx`, `NotificationsPage.jsx`, and `PageDetailPage.jsx` (×2) all get the default, cruder algorithm. Concretely: a post shows "2h," the reply directly underneath it shows "2h ago" — two different timestamp styles on the same screen, from the same component.

Mobile only ever had one algorithm (`lib/timeAgo.js`) — no "compact vs. default" split exists there, and it's used at all 8 of mobile's call sites. Web's own `formatCompact` is a manual re-implementation of it, flagged in web's own code comment: *"mirrors the app's timeAgo... Keep in sync with mobile/src/lib/timeAgo.js."* Checked both algorithms character-by-character — they currently match exactly, but "keep two files in sync by hand, forever, via a code comment" is exactly the kind of setup that silently drifts the moment one side gets edited without the other.

**Contract:** one relative-time algorithm — mobile's, since it's the one already used unconditionally everywhere with no split. Web drops `formatRelative` and the `compact` prop entirely; every `Timestamp` usage gets the same formatting. The algorithm itself should live in one place both platforms actually import — most naturally `@kowloon/client`, alongside the other cross-platform-shared logic (`resolveEmbed`, `prefs/manifest.js`, etc.) — rather than two files with a comment asking someone to remember to keep them matching.

## Platform notes

- Web's `Timestamp` optionally wraps in a `<Link>` via a `to` prop (post timestamps link to the post). Mobile's timestamps, at the 8 sites checked, are always plain text, never tappable on their own — not necessarily a gap, since the containing row is usually already tappable (a `Pressable` wrapping the whole card/row), just worth confirming during implementation that mobile isn't relying on a timestamp-specific tap target anywhere that would need porting.
- Web renders a real `<time dateTime=... title=...>` element with the full datetime as a hover tooltip — a genuine accessibility/desktop-affordance web has that mobile structurally can't (no hover). Keep as a platform difference, not something to reconcile away.

## Tokens used

- Typography: `font-ui`, uppercase, letter-spaced (chrome) — timestamps are metadata, never reader-controlled content
- Palette: `base-content` at reduced opacity (muted, matches other metadata treatments established in Card/Reply)
