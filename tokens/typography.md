# Typography tokens

Two separate systems exist today: **platform-controlled chrome typography** (fixed, same for every reader) and **reader-controlled reading typography** (a per-reader accessibility preference). They're documented separately below, along with the boundary between them.

## Chrome typography (UI text — labels, nav, buttons, headings)

| Role | Web (`kowloon-frontend`) | Mobile (`kowloon-mobile`) |
|---|---|---|
| Display | Inter | *(no separate display font)* |
| UI | IBM Plex Sans | Inter |
| Reading | Source Serif 4 | Lora |

Web exposes these as `font-display` / `font-ui` / `font-reading` Tailwind utilities. Mobile exposes `font-ui` / `font-reading` the same way via NativeWind, but mapped to different actual fonts, and has no `font-display` token at all.

**This is unreconciled drift**, not an intentional platform difference — nobody decided mobile should use Lora where web uses Source Serif 4. Worth resolving as part of the redesign pass.

## Reading typography — reader-controlled (mobile only, so far)

Mobile has a separate, more elaborate system for actual prose reading surfaces — modeled on Kindle-style reading settings, source of truth at `kowloon-mobile/src/lib/typography.js`:

- Five bundled fonts: Inter, Atkinson Hyperlegible, Lora, Merriweather, OpenDyslexic (Regular/Bold/Italic each)
- Four stepped preferences: `fontFamily`, `fontSize` (xs–xl), `lineSpacing` (compact/normal/relaxed), `columnWidth` (narrow/normal/wide)
- Account-level, synced to `user.prefs.typography` on the server — **this is a per-reader preference, applied to what that reader sees, never something an author can impose on other people's view of their post.** Same model as a Kindle: your font choice changes the book's running text on your device, not the book as the author published it.

### Where the reader's font choice applies, and where it doesn't

This is the actual Kindle-style boundary, and it's a deliberate line, not just "wherever it currently happens to be wired up":

**Applies (reader's chosen font):**
- Post, Article, and Page body text
- Reply body text — included deliberately, not just "body text of top-level posts." Two of the five bundled fonts (OpenDyslexic, Atkinson Hyperlegible) exist for real reading-disability accommodation, not taste; if a reader picked one of those, restricting it to top-level posts only would leave them hitting unreadable text in every reply thread, half-defeating the accessibility purpose.
- **Open question, not yet decided:** should the reader's font also apply to the *compose editor* while they're writing that same kind of content? Kindle has no authoring surface, so the analogy doesn't extend cleanly here. Leaning toward yes — the composer is a preview-as-you-type reading surface, and rendering it in the fixed platform font while it's about to be read back in the reader's own chosen font is a jarring mismatch — but this needs an explicit decision, not an assumption.

**Does not apply (fixed platform typography, same for every reader):**
- Post/article/page *titles* — these are platform-typeset headlines, not the prose itself (see the login-vs-post-title distinction in `IDEOLOGY.md` §1)
- All UI chrome: nav, labels, buttons, bylines, timestamps, metadata ("3 replies", etc.), settings screens, empty states

**OPEN — needs sign-off:** web has no equivalent of this system at all today. That's very likely an accident of build order (mobile shipped it first), not a real product decision — the accessibility motivation doesn't care what platform someone's reading on. Recommend web get the identical feature: same five fonts, same four preferences, same `user.prefs.typography`, same boundary rules above.
