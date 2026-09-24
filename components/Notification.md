# Notification (row)

**Purpose:** A single notification in the notifications list — actor, type, summary, timestamp, mark-read/dismiss.

## Found: third instance of the same dead-code pattern this series keeps hitting

`kowloon-frontend/src/components/notifications/NotificationItem.jsx` and `NotificationList.jsx` are unreachable — nothing outside that pair imports either one. The real, live implementation is inline in `pages/NotificationsPage.jsx` (`NotifCard`/`NotifBody`). Same shape as `ReplyList.jsx` (Reply audit) and `FollowButton.jsx`/`UserCard.jsx`/`UserList.jsx` (AddToCircle audit) — a simpler, earlier component superseded by a richer inline implementation, never deleted. Worth naming as a pattern now that it's the third occurrence: **when auditing a web component, check whether the file you found is actually the one rendering, not just the one that shares the obvious name** — `git grep` the actual page before trusting a `components/` file is live.

## Found: web still actively codes for `follow` notifications — mobile explicitly, deliberately doesn't

The real `NotifCard`/`NOTIF_ICONS`/`NOTIF_COLORS`/`FILTER_TYPES` in `NotificationsPage.jsx` all treat `follow` as a first-class, fully-supported type — its own icon (`UserPlus`), its own color (`text-success`), a filter chip, and dedicated routing logic ("*a follow (add-to-circle) points at the actor's profile*"). Mobile's `lib/notifications.js` has the opposite: `follow` is **explicitly absent**, with a comment citing the actual product policy by name — *"follow is intentionally absent — adding someone to a circle is a private act of curation in Kowloon, the followed person is never notified. See feedback_no_follow_notifications."*

This is a real, well-documented product decision (private circle curation, target never notified — same policy that made `FollowButton.jsx` dead code in the AddToCircle audit). If that policy holds server-side, a `type: "follow"` notification can never actually be generated, which means all of web's `follow` handling here is unreachable in practice even though it isn't unreachable in the import graph — a subtler kind of dead code than the first finding. **Recommend removing `follow` from web's `NOTIF_ICONS`/`NOTIF_COLORS`/`FILTER_TYPES`/routing to match mobile — but confirm server-side first that `type: "follow"` is genuinely never emitted before deleting**, since this is inference from documented policy, not a direct check of the notification-creation code path.

## Found, the other direction: web has a `moderation` type mobile doesn't handle at all

Web's `NOTIF_ICONS`/`NOTIF_COLORS` include `moderation` (`Flag` icon, `warning` color) — mobile's `NOTIF_TYPES` has no entry for it. Given real moderation-notification work shipped this cycle, a moderation notification arriving on mobile today would fall through to `NotificationRow`'s bare fallback (`{ label: notification?.type, Icon: null }`) — showing the literal string "moderation" with no icon, while web shows it properly. **Contract: add `moderation` to mobile's `NOTIF_TYPES`**, matching web's `Flag`/`warning` treatment.

## OPEN — two real judgment calls, not resolved here

- **Icon color: web rainbow-codes per type (primary/error/success/warning/secondary); mobile keeps every icon a single muted ink color.** Notifications are chrome, not content — IDEOLOGY.md §4 reserves decorative-but-meaningful color specifically for content-type wayfinding (post types), not general chrome. That argues for mobile's restraint. But a rainbow of notification-type colors is also a real, useful scanability aid in a list you're skimming for what matters. Not deciding this here — flagging both sides.
- **Unread indicator shape: web uses a small square dot (positioned after the action buttons); mobile uses a full-height 3px left-edge bar.** Pick one. Leaning toward mobile's bar, since a left-edge accent bar is already the established unread/kind-indicator shape elsewhere (Toast's kind-colored bar), but not locking that in without confirmation.

## Matched already — no action needed

- Both fade read rows to the same treatment (`opacity-60` / `opacity: 0.6`) — this one's already consistent, the dead `NotificationItem.jsx` I initially compared against wasn't representative of the real web behavior.
- Both mark-read-and-navigate on a tap/click anywhere on the row, not just an explicit button (web's `NotifCard` is even keyboard-accessible: `role="button" tabIndex onKeyDown`).
- Actor avatar shown on both.

## Platform note, not a gap

Web has an explicit "mark read without navigating away" affordance (a hover-revealed check button, since only unread rows show it) that mobile lacks — mobile can only mark read by tapping into the notification. This might be a real, worth-adding capability rather than just a hover-affordance difference (hover doesn't exist on touch, but a persistent small button could) — not resolved here, worth a deliberate decision during implementation rather than silently porting or silently skipping.

## Tokens used

- Palette: `primary` (unread indicator), `base-100`/`base-200`/`base-300` (surface, hover, dividers), plus whichever resolution the icon-color question lands on
- Typography: `font-ui` for chrome (label, timestamp, actions), the notification summary text itself is chrome too (platform-authored, not reader content) even though it's *about* content
