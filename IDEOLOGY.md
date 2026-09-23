# Kowloon Design Ideology

This is a formalization, not an invention. The aesthetic described here is already ~80% real across the codebase — it's in `kowloon-frontend/CLAUDE.md`, in the palette, in dozens of already-shipped screens. What hasn't existed until now is a written set of rules precise enough to answer "does this new component fit?" without re-deciding it from scratch. That's the actual problem the recent drift (Button's mismatched variants, two different icon libraries, two different chrome-font systems) came from: good instincts applied inconsistently because there was nothing to check against.

Where this doc makes an actual new call rather than just writing down what's already true, it's marked **OPEN — needs sign-off** and explained. Nothing marked that way is final until you say so.

## 1. What Kowloon looks like, and why

**Reference points: 1950s–60s midcentury print design, and Blue Note Records album sleeves.** Confident, restrained typography; a small fixed set of flat colors used with intent, not decoration; generous margins; illustration and photography that's cropped and bold rather than glossy or stock-feeling. The masthead treatment on the login screen — a huge "KOWLOON" wordmark over an illustrated skyline, one hairline rule, an uppercase eyebrow line — is the clearest existing example of the whole ideology in one screen.

**Why this reference, specifically, for this product:** Kowloon is a federated network built by and for people who read and write for a living, explicitly positioned against the interchangeable card-in-a-box feed aesthetic every other social app converges on. Restraint here isn't just taste — it's the same instinct as [[feedback_human_value_over_profit]] and [[feedback_no_follow_notifications]]: this app doesn't want to look or feel like it's optimizing you. A magazine doesn't pulse a red badge at you to get you to open it. Neither should this.

**The organizing principle that makes the type/color/spacing rules below cohere:** *chrome is quiet, content is confident.* Navigation, labels, buttons, and UI copy stay small, uppercase, letter-spaced, and restrained — they get out of the way. Post titles, article headlines, and body copy get the serif, the size, the weight, the actual presence. The app's job is to be a good frame for what people wrote, not to compete with it. This is already how it works — the login screen's own "Sign in to your account" heading is bold sans UI chrome, while a post title like "The Last American Christian" or "Günther Anders, the Philosopher at the End of the World" is set in the serif reading face. Nobody wrote this rule down before; it's real and it should stay real.

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

**Content vs. chrome is two separate typography systems, not one scale.** Titles/headlines are platform-typeset, fixed, same for every reader — they use the reading-chrome serif below. Body prose (post/article/page bodies, and replies) is reader-controlled: whichever of five fonts the reader personally selected, which may not be a serif at all (Inter and OpenDyslexic are both sans options). Full boundary — including the open question of whether this should extend to the compose editor, and the recommendation that web get this feature too — is in `tokens/typography.md`, not restated here.

**Chrome roles — currently drifted, needs a decision:**

| Role | Web today | Mobile today |
|---|---|---|
| Display (masthead, big UI headings) | Inter | *(no distinct role)* |
| UI (nav, labels, buttons, body chrome) | IBM Plex Sans | Inter |
| Reading (article/post serif) | Source Serif 4 | Lora |

**OPEN — needs sign-off, recommendation below:** Converge both platforms on:
- **Display + UI: Inter**, one family for both roles (distinguish display from ui by weight/size/tracking, not by swapping typefaces). Inter is already an asset on both platforms today, so this is a pure simplification — it drops IBM Plex Sans from web entirely and adds a "display" *treatment* (not a new font) to mobile.
- **Reading: Source Serif 4**, on both platforms, for chrome/headline use (not to be confused with mobile's separate user-selectable body-reading fonts, which stay as-is). This means bundling one new font file into mobile — a small, one-time asset cost — in exchange for the more deliberate, already-praised typeface ("beautiful reading type" per the original web design brief) instead of Lora, which was never a considered choice, just what was convenient.

Net effect: one fewer typeface family in the system overall (three total instead of four), and both platforms end up type-identical for the first time.

## 6. Iconography

**OPEN — needs sign-off:** web currently uses two different icon libraries inconsistently — FontAwesome in `PostComposer.jsx`, `lucide-react` in `NewPostPage.jsx`/`EditPostPage.jsx` (same icons, `faImage`/`faVideo`/`faMusic` vs `Image`/`Video`/`Music`, drawn by two different systems on different screens of the same flow). Recommend standardizing on **lucide-react** everywhere on web — it's the more recently adopted, more complete set, and matches the restrained line-icon style that already reads as "editorial" rather than "app." Mobile should use the same set (`lucide-react-native`) so an icon means the same weight/style on both platforms. This is a mechanical fix once decided — swap the FontAwesome imports in `PostComposer.jsx` for their lucide equivalents — but it's real UI work across real files, not just a docs change, so it's flagged here rather than just done.

## 7. Imagery

- Illustration (welcome/hero art): flat-color, warm-toned (ochre/teal/rust), cropped tightly, no gradients or 3D rendering — the Kowloon Walled City skyline illustration is the reference example.
- Photography (post media): shown as-shot, cropped to the layout grid, no filters or vignettes applied by the app.
- Avatars/icons that need a generated fallback (no user-uploaded image) use the existing hexagon mosaic technique ([[project_circle_mosaic_icon]]) rather than a generic initial-in-a-circle — square/hex geometry stays consistent with the no-rounded-corners rule even in generated art.

## 8. Motion

Not deeply exercised in the codebase yet, so stated as a principle rather than a catalogue of current examples: motion should feel like turning a page, not like a game UI. Prefer simple opacity/fade transitions over spring/bounce physics. Nothing should move on its own to attract attention (no auto-playing attention-grabbers, no idle pulsing) — consistent with §2's no-gamification-chrome rule.

## 9. Out of scope for this pass

Interaction patterns, navigation structure, information architecture, and UI copy/voice are deliberately not covered here — that's the next pass, once this visual language is settled. Don't read anything above as a decision about *how* screens are organized, only about how they look once organized.

## 10. Consolidated open decisions

1. Give `accent` its own hex, distinct from `error`. (§4)
2. Converge chrome typography on Inter (display+ui) / Source Serif 4 (reading), both platforms. (§5)
3. Standardize icons on lucide-react / lucide-react-native, retire FontAwesome from web. (§6)
4. Resolve `Button`'s variant/state/size drift per `components/Button.md` — recommend the *union*: keep `accent` (once §4 is resolved), add `loading` to web, add a `size` scale to mobile. Nothing gets removed, the contract gets completed.
5. Bring web up to parity with mobile's reader-controlled typography system (same fonts, same prefs, same boundary rules). (`tokens/typography.md`)
6. Decide whether reader-controlled font extends to the compose editor while writing. (`tokens/typography.md`)
