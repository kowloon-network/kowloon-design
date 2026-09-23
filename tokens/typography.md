# Typography tokens

Two separate systems: **chrome typography** (fixed, platform-controlled, same for every reader) and **content typography** (reader-controlled — an accessibility preference the reader sets once and carries everywhere they read).

## Chrome typography — unified, one face

**RESOLVED 2026-09-23:** all UI chrome — nav, buttons, labels, settings headings, bylines, timestamps, metadata — converges on **Inter**, one family, distinguished by weight/size/tracking rather than by swapping typefaces. This replaces the earlier three-role drift (web: Inter/IBM Plex Sans/Source Serif 4; mobile: Inter/Lora) and also replaces an earlier draft of this doc that proposed keeping a separate fixed serif ("reading-chrome") role for titles — see below, that role no longer exists.

Web currently exposes `font-display`/`font-ui`/`font-reading` as three Tailwind roles; mobile exposes `font-ui`/`font-reading` via NativeWind. Both collapse to a single chrome role once this ships — the multi-role tokens can go away rather than all pointing at the same font.

## Content typography — reader-controlled

Modeled on Kindle-style reading settings, currently mobile-only, source of truth at `kowloon-mobile/src/lib/typography.js`:

- Five bundled fonts: Inter, Atkinson Hyperlegible, Lora, Merriweather, OpenDyslexic (Regular/Bold/Italic each)
- Four stepped preferences: `fontFamily`, `fontSize` (xs–xl), `lineSpacing` (compact/normal/relaxed), `columnWidth` (narrow/normal/wide)
- Account-level, synced to `user.prefs.typography` on the server — a per-reader preference applied to what that reader sees, never something an author can impose on other people's view of their post. Your font choice changes the book's running text on your device, not the book as the author published it.

**RESOLVED 2026-09-23**, confirmed against a live visual comparison: post/article/page **titles are also reader-controlled**, in the same family as the body, differentiated from body text by size/weight only (the way a real headline and its running text share a typeface family in print). This is a change from an earlier draft that gave titles a fixed platform serif — reconsidered because a title is authored content the reader is about to read, same as the body under it, not platform chrome.

**Applies (reader's chosen font, size/weight varies by role):**
- Post, Article, and Page titles *and* body text — same family, title bigger/bolder
- Reply body text — deliberately included, not just top-level posts. Two of the five bundled fonts (OpenDyslexic, Atkinson Hyperlegible) exist for real reading-disability accommodation, not taste; restricting them to top-level posts only would leave a reader who needs one hitting unreadable text in every reply thread.
- **RESOLVED 2026-09-23:** the compose editor, while writing that same kind of content — confirmed rather than left open. Rendering the composer in a different font than what the reader will see when they read it back was the jarring-mismatch concern; extending the reader's font here avoids that.

**Does not apply (fixed chrome — Inter):**
- Nav, labels, buttons, settings screens, bylines, timestamps, metadata ("3 replies," etc.), empty states — anything the platform itself authored rather than the post's author

**RESOLVED 2026-09-23:** web gets this feature too — same five fonts, same four preferences, same `user.prefs.typography`, same boundary above. Not building it as a mobile-only accommodation was an accident of build order, not a decision.

## Visual check

Confirmed 2026-09-23 — Josh reviewed a live comparison (the real "The Last American Christian" post, title and body rendered in all five candidate fonts, real chrome around it) and approved the approach as a whole. No single font was chosen as a default; the point confirmed was the *mechanism* (reader-controlled title+body, same family, size/weight differentiation) — all five fonts remain available reader choices, same as today on mobile.
