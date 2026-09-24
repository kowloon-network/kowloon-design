# PostCard

**Purpose:** The feed preview card — the single most-repeated composite in the app. Renders a post's author row, type-aware body, media, reactions, and action bar. Also covers `EventCard`, since Event gets first-class distinct treatment.

## Structural difference, not just style drift

Web decomposes into five real components: `PostMeta` (author row + type/time), `PostBody` (title/body/media, shared with the full post detail page via a `showFull` prop), `PostReacts`, `PostToolbar`, and `EventCard` as a fully separate component. Mobile is one 413-line `PostCard.jsx` handling every post type's rendering inline via branching — and mobile has a *second*, separate, unshared implementation of "render title/body/media" in its own `PostBody.jsx`, used only by the detail screen, not by the feed card. That's worth flagging on its own: mobile has the same "two independent copies of the same rendering logic" problem this whole audit keeps finding elsewhere, just within one platform this time rather than across both. Not addressed in this pass — real follow-up work once Card implementation starts.

## RESOLVED 2026-09-24 — four decisions from this session

### 1. Fix: byline must not use the reader's chosen font

Mobile's `PostCard.jsx` applies `resolved.boldFamily`/`resolved.regularFamily` (the reader's typography preference) to the author's name and handle, via `nameStyle`/`handleStyle`. That directly violates the already-resolved typography boundary (`tokens/typography.md`): bylines are fixed chrome, not reader-controlled content, regardless of what font the reader picked for reading. Web's `PostMeta.jsx` gets this right structurally (`font-ui`, not wired to any reading preference) — mobile needs the same fix. Author name and handle render in fixed Inter, same as every other byline/metadata/timestamp on the card.

### 2. Media: mobile's grid, capped at 4 (2×2) in the feed card, with a "see more" link

Josh's call: mobile's 2-column image grid is the preferred style over web's main-image-plus-thumbnail-strip pattern — adopt it on both platforms. But **neither platform currently caps attachment count in the feed card today** — both show every attachment, unbounded. New behavior, not a fix: the feed card shows at most 4 images (2×2); a 5th+ attachment replaces the grid's bottom-right cell with a "+N more" / "see more" link to the full post, matching the existing `ContinueReading` pattern already used for truncated text. Video/audio attachments (mobile currently renders these as separate full-width rows below the image grid) count toward the same cap and logic.

**Open, not addressed here:** whether the full post detail page's media view should also switch to the capped grid style, or keep web's richer main+thumbnail-strip+lightbox experience there. Josh scoped his preference to "the feed post card" specifically — the detail-page experience is a separate question.

### 3. Event: adopt web's calendar-block treatment on both platforms

Josh's call: web's `EventCard` — a calendar tear-off block (month + day, styled like a torn desk-calendar page) beside the title, plus a start-time/location subheader — is the version to keep, on both platforms. This is genuinely new, not a fix, for mobile: it currently has *no* distinct Event treatment at all, rendering Events through the same generic branch as Note/Article with only a `prominent` flag on the location line. Mobile needs to build the calendar block and subheader from scratch. Web's implementation (`EventCard.jsx`) is the reference to port from — `CalendarBlock` (month strip in `post-event` color, day number in display type, day-of-week below), `subheader` (start time + location joined with `|`).

### 4. Remove the hairline rules above/below the action bar, on both platforms — located precisely on web

Josh's screenshot (from the web app in a phone browser, not the native app) pinned down exactly what "feels cluttered": web's `PostCard.jsx` has a literal `border-t border-base-300` on the icon/toolbar row (`<div className="flex items-center gap-3 pt-2 border-t border-base-300">`, wrapping `VisibilityIcon` + `PostTypeIcon` + `PostToolbar`) and a `border-b border-base-300` on the card's own outer `<article>` wrapper — that second one is what reads as a rule directly below the action row, since the action row is the last thing inside the card before that border kicks in. Same two classes exist verbatim in `EventCard.jsx`. **Confirmed fix for web:** drop both border classes; add a bottom margin (~one line-height, e.g. `mb-4`/similar) to the card in their place, so posts still get breathing room without a rule line.

**Mobile does not have an equivalent literal border in the source** this audit traced (`PostActionBar`, `ReactsBar`/`ReactSummaryRow`, and their wrapping `View`s on both the feed card and the detail screen) — so mobile may already be close to the target state. Josh confirmed the fix applies to both platforms regardless (for visual consistency, same spacing amount below each card either way): mobile should get the same ~one-line-height bottom margin web gets, and double-check no border-like rule shows up anywhere in that region once implemented, even though none was found in this pass.

## Tokens used

- Palette: `post-note`/`post-article`/`post-media`/`post-link`/`post-event` (type label + Event calendar-block color), `base-300` (dividers), `base-content` at various opacities (metadata)
- Typography: `font-ui` for all chrome (byline, type label, timestamp, "see more"/"Continue reading" links), reader-controlled font for title/body only, per the resolved boundary
