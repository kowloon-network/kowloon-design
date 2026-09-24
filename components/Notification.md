# Notification (row)

**Purpose:** A single notification in the notifications list — actor, type, summary, timestamp, mark-read/dismiss.

## Found: third instance of the same dead-code pattern this series keeps hitting

`kowloon-frontend/src/components/notifications/NotificationItem.jsx` and `NotificationList.jsx` are unreachable — nothing outside that pair imports either one. The real, live implementation is inline in `pages/NotificationsPage.jsx` (`NotifCard`/`NotifBody`). Same shape as `ReplyList.jsx` (Reply audit) and `FollowButton.jsx`/`UserCard.jsx`/`UserList.jsx` (AddToCircle audit) — a simpler, earlier component superseded by a richer inline implementation, never deleted. Worth naming as a pattern now that it's the third occurrence: **when auditing a web component, check whether the file you found is actually the one rendering, not just the one that shares the obvious name** — `git grep` the actual page before trusting a `components/` file is live.

## Found: web still actively codes for `follow` notifications — mobile explicitly, deliberately doesn't

The real `NotifCard`/`NOTIF_ICONS`/`NOTIF_COLORS`/`FILTER_TYPES` in `NotificationsPage.jsx` all treat `follow` as a first-class, fully-supported type — its own icon (`UserPlus`), its own color (`text-success`), a filter chip, and dedicated routing logic ("*a follow (add-to-circle) points at the actor's profile*"). Mobile's `lib/notifications.js` has the opposite: `follow` is **explicitly absent**, with a comment citing the actual product policy by name — *"follow is intentionally absent — adding someone to a circle is a private act of curation in Kowloon, the followed person is never notified. See feedback_no_follow_notifications."*

This is a real, well-documented product decision (private circle curation, target never notified — same policy that made `FollowButton.jsx` dead code in the AddToCircle audit). **RESOLVED 2026-09-24 — Josh's call: remove it from web.** Drop `follow` from `NOTIF_ICONS`/`NOTIF_COLORS`/`FILTER_TYPES`/routing to match mobile. One implementation-time sanity check, not a reversal of the decision: confirm server-side that `type: "follow"` is genuinely never emitted before deleting the routing branch specifically (the one piece of this that would silently misbehave, rather than just look wrong, if the assumption turned out incomplete) — the decision to remove stands either way.

## Found, the other direction: web has a `moderation` type mobile doesn't handle at all

Web's `NOTIF_ICONS`/`NOTIF_COLORS` include `moderation` (`Flag` icon, `warning` color) — mobile's `NOTIF_TYPES` has no entry for it. Given real moderation-notification work shipped this cycle, a moderation notification arriving on mobile today would fall through to `NotificationRow`'s bare fallback (`{ label: notification?.type, Icon: null }`) — showing the literal string "moderation" with no icon, while web shows it properly. **Contract: add `moderation` to mobile's `NOTIF_TYPES`**, matching web's `Flag`/`warning` treatment.

## RESOLVED 2026-09-24: two independent signals, not one shape doing double duty

Josh's call splits what was previously conflated in each platform's single treatment:

- **Unread state → the dot.** Web's existing small square dot becomes the standard on both platforms. Mobile's current left-edge bar-for-unread goes away — that role is now the dot's alone.
- **Notification type → a left-edge color bar**, echoing the same pattern `Toast.md` already established (a left-edge bar colored by kind). This is genuinely new — neither platform's current code does exactly this; web currently colors the *icon* itself per type, mobile doesn't color-code type at all. **The icon itself stays a single muted ink color on both platforms** — the bar carries the color signal, so the icon doesn't also need to, which keeps the row from getting two competing color hits and matches mobile's existing restraint for the icon glyph specifically. Bar color mapping reuses web's existing `NOTIF_COLORS` (`reply`→primary, `react`→error, `moderation`→warning, `join_request`/`join_approved`→secondary, `new_post`→muted/base-content) minus `follow`, which is removed per the decision above.

Net result: a row's left edge tells you *what kind* of notification it is at a glance (color bar), independent of a small dot telling you whether you've *seen* it yet — two signals, neither one overloaded to carry both meanings.

## Matched already — no action needed

- Both fade read rows to the same treatment (`opacity-60` / `opacity: 0.6`) — this one's already consistent, the dead `NotificationItem.jsx` I initially compared against wasn't representative of the real web behavior.
- Both mark-read-and-navigate on a tap/click anywhere on the row, not just an explicit button (web's `NotifCard` is even keyboard-accessible: `role="button" tabIndex onKeyDown`).
- Actor avatar shown on both.

## Platform note, not a gap

Web has an explicit "mark read without navigating away" affordance (a hover-revealed check button, since only unread rows show it) that mobile lacks — mobile can only mark read by tapping into the notification. This might be a real, worth-adding capability rather than just a hover-affordance difference (hover doesn't exist on touch, but a persistent small button could) — not resolved here, worth a deliberate decision during implementation rather than silently porting or silently skipping.

## Tokens used

- Palette: `primary` (unread dot), `primary`/`error`/`warning`/`secondary`/`base-content` (type color bar, per `NOTIF_COLORS`), `base-100`/`base-200`/`base-300` (surface, hover, dividers), `base-content` at reduced opacity (muted icon, label, timestamp)
- Typography: `font-ui` for chrome (label, timestamp, actions), the notification summary text itself is chrome too (platform-authored, not reader content) even though it's *about* content
