# Typography tokens

Two separate systems exist today, documented here as-is rather than forced into one — reconciling them is a real design decision the redesign pass should make deliberately, not something to paper over.

## Chrome typography (UI text — labels, nav, buttons, headings)

| Role | Web (`kowloon-frontend`) | Mobile (`kowloon-mobile`) |
|---|---|---|
| Display | Inter | *(no separate display font)* |
| UI | IBM Plex Sans | Inter |
| Reading | Source Serif 4 | Lora |

Web exposes these as `font-display` / `font-ui` / `font-reading` Tailwind utilities. Mobile exposes `font-ui` / `font-reading` the same way via NativeWind, but mapped to different actual fonts, and has no `font-display` token at all.

**This is unreconciled drift**, not an intentional platform difference — nobody decided mobile should use Lora where web uses Source Serif 4. Worth resolving as part of the redesign pass.

## Reading typography (mobile only, so far)

Mobile has a separate, more elaborate system for the actual post/article reading surface (not chrome) — modeled on Kindle-style reading settings, source of truth at `kowloon-mobile/src/lib/typography.js`:

- Five bundled fonts: Inter, Atkinson Hyperlegible, Lora, Merriweather, OpenDyslexic (Regular/Bold/Italic each)
- Four stepped preferences: `fontFamily`, `fontSize` (xs–xl), `lineSpacing` (compact/normal/relaxed), `columnWidth` (narrow/normal/wide)
- Account-level, synced to `user.prefs.typography` on the server

Web has no equivalent user-configurable reading typography today — it just renders body text in the chrome reading font. Whether that's an intentional web/mobile difference (reading settings matter more on a phone) or something web should eventually get is an open question, not a decision made here.
