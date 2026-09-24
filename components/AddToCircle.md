# AddToCircle

**Purpose:** The "follow" action, Kowloon-style — adding a person to one of the viewer's own circles. Opens a picker: search your circles, tap to add, inline checkmark for ones that already contain this person.

## Found: dead code implementing the wrong product model

`kowloon-frontend/src/components/users/FollowButton.jsx` is a real, working `isFollowing`/`onFollow`/`onUnfollow` toggle — literal "Follow"/"Unfollow" button text, `i18n` keys `common.follow`/`common.unfollow`. That directly contradicts the actual, documented product model (mobile's own `ProfileActions.jsx` states it plainly: *"Kowloon has no follow/unfollow — adding a person to one of your circles IS the follow... the target is never notified"*). Traced the whole chain: `FollowButton.jsx` is only used by `components/users/UserCard.jsx`, which is only used by `components/users/UserList.jsx`, which **nothing imports anywhere in the codebase.** All three files are fully unreachable dead code — a legacy pre-circles-model implementation nobody deleted when the product pivoted. Flag for deletion during implementation, not something to spec a contract for.

(The live, actually-used-in-a-real-page `UserCard` on `UsersPage.jsx` is a separate, locally-scoped function of the same name — a directory-listing row with no follow/add action at all, correctly just linking to the profile. Not broken, just an unfortunate naming collision with the dead file.)

## The real implementation is already well-converged — Field/TypeFilter-shaped, not Card-shaped

Web's real component, `AddToCircleButton.jsx`, opens its own comment with *"matching the mobile app's profile Add-to-Circle flow"* — and it does: same server-side membership seeding (`?contains`), same sort-by-pins ordering, same search-your-circles box, same inline checkmark, same disable-once-added logic. This is core-logic-already-shared territory, same as `TypeFilter`.

## RESOLVED: feedback treatment — a direct consequence of the Toast contract

Web shows `toast.success()` on a successful add, with a "View" action linking straight to the circle; `toast.error()` on failure. Mobile has no success feedback at all (only the inline checkmark updates) and uses `Alert.alert("Couldn't add", ...)` for errors — exactly the kind of single-message, no-decision-required feedback `components/Toast.md` already identified as needing to move off `Alert.alert`. **Contract: once mobile's toast system exists, `ProfileActions.jsx` gets the same success/error toast treatment as web, including the "View circle" action link.** Not a new decision — this is `Toast.md`'s resolution applied to a specific, concrete site.

## Found: web's dialog doesn't reuse the shared `Modal.jsx` — and has better behavior than it

`AddToCircleButton.jsx` hand-rolls its own `createPortal` overlay (backdrop + dialog markup) instead of using `components/ui/Modal.jsx`. It's not obviously wrong to do that — it has real behavior `Modal.jsx` lacks: ESC-to-close, body-scroll lock while open, and a responsive layout (full-height inset on narrow viewports, centered `max-w-md` on wide ones) that's more considered than `Modal.jsx`'s fixed `max-w-lg`. **Worth feeding back into `Modal.md`**: these three behaviors (ESC-to-close, scroll-lock, responsive sizing) should probably become part of the shared `Modal` primitive's own contract, rather than staying a one-off improvement only this component has.

Mobile's picker uses a full-screen `Modal` (not the bottom-`Sheet` convention `components/Modal.md` established) — but for a documented, legitimate reason: *"a slide-up sheet clipped its last row under Android's nav bar."* Treat as an accepted exception to the Sheet convention, not drift — a real engineering constraint, not an arbitrary choice.

## Platform notes

- Mobile bundles Block/Mute into the same component (`ProfileActions.jsx`, an overflow "..." menu next to Add to Circle). Web's equivalent Block/Mute UI wasn't traced in this audit — worth checking during implementation whether it lives elsewhere on the profile page with the same anchored-dropdown-menu shape, which would be a fourth overlay pattern (anchored near its trigger, neither centered `Modal` nor bottom `Sheet`) worth a note in `Modal.md` if confirmed.

## Tokens used

- Palette: `primary`/`primary-content` (Add to Circle button), `success` (added checkmark, matches web's `text-success`), `base-100`/`base-200`/`base-300` (dialog surface, rows, dividers)
- Typography: `font-ui` throughout (chrome — this is a utility picker, not reader-controlled content)
