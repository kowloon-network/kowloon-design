# Kowloon Design Ideology

This is a formalization, not an invention. The aesthetic described here is already ~80% real across the codebase — it's in `kowloon-frontend/CLAUDE.md`, in the palette, in dozens of already-shipped screens. What hasn't existed until now is a written set of rules precise enough to answer "does this new component fit?" without re-deciding it from scratch. That's the actual problem the recent drift (Button's mismatched variants, two different icon libraries, two different chrome-font systems) came from: good instincts applied inconsistently because there was nothing to check against.

Where this doc makes an actual new call rather than just writing down what's already true, it's marked **OPEN — needs sign-off** and explained. Nothing marked that way is final until you say so.

## 1. What Kowloon looks like, and why

**Reference points: 1950s–60s midcentury print design, and Blue Note Records album sleeves.** Confident, restrained typography; a small fixed set of flat colors used with intent, not decoration; generous margins; illustration and photography that's cropped and bold rather than glossy or stock-feeling. The masthead treatment on the login screen — a huge "KOWLOON" wordmark over an illustrated skyline, one hairline rule, an uppercase eyebrow line — is the clearest existing example of the whole ideology in one screen.

**Why this reference, specifically, for this product:** Kowloon is a federated network built by and for people who read and write for a living, explicitly positioned against the interchangeable card-in-a-box feed aesthetic every other social app converges on. Restraint here isn't just taste — it's the same instinct as [[feedback_human_value_over_profit]] and [[feedback_no_follow_notifications]]: this app doesn't want to look or feel like it's optimizing you. A magazine doesn't pulse a red badge at you to get you to open it. Neither should this.

**The organizing principle that makes the type/color/spacing rules below cohere:** *chrome is quiet, content is the reader's own.* Navigation, labels, buttons, and UI copy stay small, uppercase, letter-spaced, restrained, and fixed — one typeface (Inter), the same for everyone, that gets out of the way. Post titles and body copy get the size, the weight, the actual presence — and, as of the type-system decision in §5, they're set in whatever typeface the *reader* has chosen for their own reading, not a typeface the platform imposes. The app's job is to be a good, quiet frame for what people wrote and how each reader wants to read it — not to compete with either. The login screen's own "Sign in to your account" heading is fixed chrome; a post title like "The Last American Christian" is not chrome at all, it's the first line of what the reader is here to read.

## 2. Non-negotiables

These are the checkable rules — if a new component breaks one of these, that's a bug, not a style choice.

1. **No rounded corners, anywhere.** Square corners on every surface: buttons, cards, inputs, images, avatars, modals. This is already the rule on both platforms; keep it absolute, no "just this once for a pill badge" exceptions.
2. **No drop shadows or elevation blur.** Separation between surfaces comes from a hairline rule, whitespace, or a flat color block — never a soft shadow. Shadows read as "app," and the whole point is not reading as an app.
3. **No gamification chrome.** No streaks, no pulsing badges, no follower counts presented as a score. This is already policy for circles/follows ([[feedback_no_follow_notifications]]); it's a design rule as much as a product one — if a feature doesn't want the mechanic, the UI shouldn't imply it either.
4. **Uppercase is for fixed platform chrome only — never for content whose exact characters matter.** Nav items, button labels, and form field labels (the word "USERNAME," not what's typed into it) are a small, fixed, platform-authored vocabulary and can be uppercase and letter-spaced — this is still the app's clearest visual signature. But uppercase silently destroys information: it erases the distinction between a proper noun and a common word, hides which letters in a handle or acronym are actually capitalized, and is measurably harder to read at length because it removes word-shape recognition — directly at odds with the accessibility intent behind the reading-typography system (§5). So it never applies to: usernames, handles, bylines, emails, URLs, or placeholder/example text that's demonstrating a real format. (Concrete existing bug: the login screen's username placeholder — meant to show the exact `@you@example.com` shape — is currently forced into caps, which actively obscures the thing it's supposed to demonstrate.) **Revised from an earlier, too-broad version of this rule** that would have uppercased bylines and any label regardless of content — corrected 2026-09-23.
5. **Chrome is sans, content is serif — never inverted.** UI headings, buttons, labels, navigation: sans (display or ui role). Post titles, article headlines, post/page body copy: serif (reading role). A screen that puts a serif face on a settings-page heading, or a sans face on a post title, is wrong.
6. **Color is semantic, not decorative.** Every color on screen should be traceable to a role (see §4) — a state, a content type, an emphasis level. No color exists purely to make a screen "feel less blank." If a screen feels blank, that's a whitespace/type-scale problem, not a color problem.
7. **Whitespace is generous by default.** When in doubt, more margin, not less. This is a print-page instinct, not a mobile-app-density instinct — resist the pull toward cramming more into the viewport.

## 3. Layout rhythm

- **12-column grid**, sidebar(3) : main(6) : sidebar(3) on web, and the same 3:6:3 split on landscape tablet via `TabletColumns` on mobile ([[project_mobile_tablet_layout]]).
- **`gap-16`, not `gap-10`**, wherever the grid is replicated for fixed overlays — this was a real, already-hit bug ([[project_grid_replication_technique]]); it's a rule now, not a one-off fix.
- Prefer `fixed` over `sticky` for anything that needs to track scroll inside a constrained container — `sticky` + `h-0` + negative margins has already broken bottom offsets once.

## 4. Color system

The palette (`tokens/palette.json`) is already good and shouldn't change in hue — what's missing is stated intent per role, so "which color do I reach for" has an answer:

| Role | Use for |
|---|---|
| `header` (deep blue) | The one fixed brand color — masthead/header bar only. Not a general-purpose accent. |
| `primary` (steel blue) | Default interactive color — primary buttons, links, active states. |
| `secondary` (deep navy-plum) | Secondary emphasis — secondary buttons, the "Blue Note" plum accent. |
| `accent` (rose red) | Sparing, high-emphasis-only decoration — **see open issue below, this role currently collides with `error`.** |
| `neutral` / `base-*` | Structure: backgrounds, borders, body text. The vast majority of every screen should be these, by area. |
| `info`/`success`/`warning`/`error` | System feedback only. Never reused for anything else. |
| `post-note`/`post-article`/`post-media`/`post-link`/`post-event` | Content-type wayfinding — the one place color is allowed to be purely decorative-but-meaningful, like a magazine's section colors. Keep this the *only* place arbitrary-feeling color coding happens. |

**OPEN — needs sign-off: `accent` (`#c0394a`) and `error` (`#c0394a`) are the exact same hex value.** That means anything styled as "accent" for emphasis is visually indistinguishable from an error state — a real correctness risk, not just a taste question (imagine a highlighted/featured post card reading as if something broke). Recommend giving `accent` its own distinct value, separate from `error`, before it's used anywhere that could be confused with a system-error signal.

## 5. Type system

**RESOLVED 2026-09-23.** Content and chrome are two separate typography systems, not one scale, and each is now down to as few faces as it can be:

- **Chrome — one unified face, Inter, both platforms.** Nav, labels, buttons, settings screens, bylines, timestamps, metadata. Distinguish weight (display-scale headings vs. small UI labels) by size/weight/tracking, not by swapping typefaces. This drops IBM Plex Sans from web and Lora-as-chrome from mobile — both converge on the one font that was already an asset on both platforms.
- **Content — reader-controlled, both platforms.** Post/article/page titles *and* bodies, plus replies, plus the compose editor while writing them: all set in whichever of five fonts (Inter, Atkinson Hyperlegible, Lora, Merriweather, OpenDyslexic) the individual reader has chosen — title and body share the family, differing only by size/weight, the way a printed headline and its running text do. Confirmed 2026-09-23 against a live visual comparison (real post, all five fonts, real chrome) — see `tokens/typography.md`. This is a change from an earlier draft of this doc that gave titles a fixed platform serif — reconsidered because a title is authored content the reader is about to read, not something the platform itself is saying.

There is no longer a "reading-chrome" role distinct from either of the above — the three/four-typeface system that existed across both platforms collapses to two roles total. Full detail, including the compose-editor and web-parity decisions, is in `tokens/typography.md`.

## 6. Iconography

**RESOLVED 2026-09-23:** web currently uses two different icon libraries inconsistently — FontAwesome in `PostComposer.jsx`, `lucide-react` in `NewPostPage.jsx`/`EditPostPage.jsx` (same icons, `faImage`/`faVideo`/`faMusic` vs `Image`/`Video`/`Music`, drawn by two different systems on different screens of the same flow). Standardizing on **lucide-react** everywhere on web — it's the more recently adopted, more complete set, and matches the restrained line-icon style that already reads as "editorial" rather than "app." Mobile uses the same set (`lucide-react-native`) so an icon means the same weight/style on both platforms.

This is a real code change, not a docs-only decision — swap the FontAwesome imports in `PostComposer.jsx` for their lucide equivalents — but it's a mechanical, low-ambiguity one, not a taste call, so it's resolved here rather than held open. Not yet executed in `kowloon-frontend` (this repo doesn't touch other repos' code, per the no-wiring-yet rule); tracked as implementation work for whenever the component pass reaches Button/composer icons.

## 7. Imagery

- Illustration (welcome/hero art): flat-color, warm-toned (ochre/teal/rust), cropped tightly, no gradients or 3D rendering — the Kowloon Walled City skyline illustration is the reference example.
- Photography (post media): shown as-shot, cropped to the layout grid, no filters or vignettes applied by the app.
- Avatars/icons that need a generated fallback (no user-uploaded image) use the existing hexagon mosaic technique ([[project_circle_mosaic_icon]]) rather than a generic initial-in-a-circle — square/hex geometry stays consistent with the no-rounded-corners rule even in generated art.

## 8. Motion

Not deeply exercised in the codebase yet, so stated as a principle rather than a catalogue of current examples: motion should feel like turning a page, not like a game UI. Prefer simple opacity/fade transitions over spring/bounce physics. Nothing should move on its own to attract attention (no auto-playing attention-grabbers, no idle pulsing) — consistent with §2's no-gamification-chrome rule.

## 9. Out of scope for this pass

Interaction patterns, navigation structure, information architecture, and UI copy/voice are deliberately not covered here — that's the next pass, once this visual language is settled. Don't read anything above as a decision about *how* screens are organized, only about how they look once organized.

## 10. Consolidated open decisions

1. **Only real open item:** give `accent` its own hex, distinct from `error`. Three candidates under review (see visual comparison, shared 2026-09-23). (§4)
2. ~~Icon library~~ — **RESOLVED**, lucide-react/lucide-react-native, both platforms. (§6)
3. ~~Button contract~~ — **RESOLVED** (union of both platforms' capabilities; `accent` variant blocked on #1 above). (`components/Button.md`)
4. ~~Type system~~ — **RESOLVED**, see §5 and `tokens/typography.md`.
