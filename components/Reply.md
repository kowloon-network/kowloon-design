# Reply

**Purpose:** A single reply row within a post's two-level threaded discussion — avatar, byline, body, and a small action row (react, reply, edit, delete).

## Good news first: both platforms already agree more than they disagree here

Unlike Card, both platforms have one real `Reply` component, and the live threading structure is genuinely close: web (`PostPage.jsx` + `Reply.jsx`) and mobile (`Reply.jsx` self-recursing via a `childReplies` prop) both render first-level replies with second-level children indented under a left border (web: `ml-6 md:ml-11 pl-4 border-l-2 border-base-300`; mobile: `ml-11 pl-3 border-l border-base-content/10` — close enough to be a minor reconciliation, not a structural fix).

## RESOLVED / confirmed as a real gap: reply bodies aren't reader-controlled on either platform yet

Neither platform currently wires the reader's chosen typography into reply bodies. Mobile hardcodes `fontSize={14} lineHeight={20}` on its `HtmlContent`; web uses a static `font-reading text-[13.5px]` class. Per the already-resolved typography boundary (`tokens/typography.md`), reply bodies are explicitly in scope for reader-controlled font — that was the specific reasoning for including replies at all (a reader who picked OpenDyslexic/Atkinson Hyperlegible for accessibility reasons shouldn't hit a wall of fixed-font text in every reply thread). This isn't drift to reconcile, it's a real feature gap to build on both platforms, same shape as `PostBody`'s title/body wiring.

Byline (author name + timestamp) correctly stays fixed chrome on both platforms already — no bug here, unlike `PostCard`'s byline issue.

## Found: `ReplyList.jsx` is dead code on web

It's unused anywhere in the web codebase (grep confirms nothing imports it) — and it would actually be broken if something did: it expects a flat array of raw replies and renders each through `PostCard`, but the real reply-tree builder (`lib/replyTree.js`'s `buildReplyTree`, shared by `PostPage.jsx` and `ReplyModal.jsx`) returns `{ reply, children }` tree nodes, and the actual live rendering path is `PostPage.jsx` calling `<Reply>` directly with proper nesting. Not part of this contract to fix — flagging for deletion during implementation, since dead code that no longer matches the real data shape is worth removing, not maintaining.

## OPEN — needs Josh's call: should the hairline rule between reply rows also go?

Web's `Reply.jsx` has `border-b border-base-300 last:border-b-0` separating consecutive replies — the same kind of hairline rule just removed from `PostCard`/`EventCard` for reading as cluttered. Not assuming the same call applies here without asking: a threaded reply list is arguably a different context than a feed of independent posts (tighter visual grouping might read as "one conversation" rather than clutter), but it might also be the same problem. Flagging rather than deciding.

## Variants / States

| State | Contract |
|---|---|
| default | Avatar + byline + body + action row |
| editing (author only) | Body replaced by a textarea; action row replaced by Cancel/Save |
| saving / deleting | Save or Delete label shows a busy state; both platforms already handle this |
| error | Inline error message below the row; both platforms already handle this |

## Platform notes

- **Edit/Delete affordance differs on purpose, not drift:** web uses icon buttons (`Pen`/`Trash`, with `title`/`aria-label`), mobile uses uppercase text labels ("Edit"/"Delete"). This reads as a legitimate platform difference rather than something to reconcile — icons rely on a hover affordance to be discoverable, which desktop has and touch doesn't, so text labels are the more accessible choice on mobile specifically. Recommend keeping this as a stated platform difference (like Button's `children`/`label`), not forcing one style onto both.
- Second-level reply children never show their own Reply/Edit/Delete-triggering "Reply" affordance (`showReply={false}` passed to children on both platforms) — consistent with the 2-level cap, not a gap.

## Tokens used

- Palette: `base-300` (dividers/indent border — pending the open question above), `base-content` at various opacities (byline, muted labels), `primary` (edit-mode Save button), `error` (delete label, error text)
- Typography: `font-ui` for byline/labels (chrome), reader-controlled font for body once built (see gap above)
