# Abdullah's World (عالم عبدالله) — handover for Claude Code

This folder is a finished, self-contained web product for **Raneen Village**: a home screen plus three bilingual (Arabic/English) 3D learning games for ages 3–7, starring the character Abdullah. Dee (the owner) plans to sell it as a 12 KD bundle on **raneenvillage.com**. Your job is to get it live there and keep it working.

## What's in the folder

| File | What it is |
|---|---|
| `index.html` | The home screen ("Abdullah's World"): language switch, the three games, a combined sticker book, and a grown-ups progress report. |
| `star-hop.html` | **Star Hop / قفزة النجوم** — 3D platformer. Collect the whole alphabet on each island level. Easy / Medium / Hard, little-kids mode, safety net. |
| `letter-kart.html` | **Letter Kart / سباق الحروف** — kart racing. Easy: collect letters. Medium: picture → first letter + short spelling. Hard: long words. Power-ups, puddles, jumps, sticker album. Three worlds (beach, desert, city at night). |
| `letter-glider.html` | **Letter Glider / طيران الحروف** — paraglider over Kuwait in three legs. Alphabet in order, "feed the baby bird" phonics, spelling, sky-writing, postcards with heritage facts, parent report, adaptive repetition of missed letters. |
| `lib/` | three.js r128 and its BufferGeometryUtils (MIT, licence included). Nothing loads from a CDN. |
| `fonts/` | Baloo Bhaijaan 2 (Arabic + Latin, weights 500/700/800, SIL OFL, licence included). |
| `icons/`, `manifest.webmanifest`, `sw.js` | Install-to-home-screen and offline play (service worker caches every file on first visit). |

Every game is a **single HTML file** with its own JS/CSS inline, including Abdullah's artwork as data URIs. There is no build step and no backend.

## Deploy

1. Look at how `raneen-village-platform` (GitHub: `fmcclues-creator/raneen-village-platform`) serves static files before choosing a location. A path like `/abdullahs-world/` works; keep the whole folder together because all links are relative.
2. Serve over **HTTPS** (needed for the offline service worker). Correct MIME types: `.webmanifest` → `application/manifest+json`, `.woff2` → `font/woff2`.
3. `sw.js` names its cache after a hash of every file, so any rebuilt file automatically reaches returning players. If you edit files by hand instead of rebuilding, change the `CACHE` string in `sw.js` yourself.
4. Check: open `/abdullahs-world/` on a phone, play a game, turn on airplane mode, reload — it should still work.

## How the pieces talk to each other

- **Arabic is the default everywhere.** Every page starts in Arabic each time it is opened; if the child switches to English, that choice is kept in `sessionStorage` (`abdullahworld.lang`) only for the current visit, so it carries between the home screen and the games and resets to Arabic next time.
- The home screen opens games with `?hub` (e.g. `letter-glider.html?hub`). In that mode each game shows an "Abdullah's World" button.
- Opened without `?hub`, a game works on its own (that's how the claude.ai previews run).
- All progress is in `localStorage`, per device and browser, no accounts:
  - `starhop.v3` — `{ lang, kids, done: [bool, bool, bool] }`
  - `letterkart.v2` — `{ lang, diff, stickers: { en: {word:1}, ar: {word:1}, ach: {id:1} } }`
  - `letterglider.v1` — `{ lang, diff, abc: {en, ar} (next letter to play), abcCount: {en, ar} (letters caught in order, ever), miss: {en: {letter: count}, ar: {...}}, post: {id:1}, pics: {en:{}, ar:{}}, sky: {"arب":1} }`
- The home screen only reads these keys; it never writes game progress.

## Selling / access (not built yet — decide with Dee)

The folder is open to anyone who has the link. Options, simplest first:
1. Put the folder behind the platform's existing login and only show it to buyers.
2. Unlock with a code delivered by the Shopify order (store the unlock flag in `localStorage`, check it on `index.html`).

Cross-device progress would need a backend endpoint; the `localStorage` shapes above are what to sync.

## Voices

Everything a child hears is a real recording; the browser's speech voice is only a fallback if a file is missing.
- `audio/ar/letters/NN-name.mp3`: Dee's own 32 Arabic letter recordings (byte-for-byte originals).
- Everything else was generated in Dee's ElevenLabs account. Arabic is Standard Arabic (فصحى) with full vowel marks, voice **3ain** (Abdullah's name is written «عَبْدَالله» so it is said "Abdallah"). An earlier take in the voice Hasan is kept on Dee's Mac in hasan_backup. English uses **Jeanette**. Abdullah's own little reactions use **character male young**.
  - `audio/<ar|en>/lines/`: praise (9, picked at random), try-again (5), countdown, and each game's event lines (power-ups, landings, checkpoints...).
  - `audio/<ar|en>/words/`: every picture word in Letter Kart and Letter Glider. Arabic files are named by English meaning (حقيبة → `bag.mp3`).
  - `audio/<ar|en>/postcards/`: the 7 Kuwait postcards.
  - `audio/en/letters/a.mp3`...`z.mp3`; `audio/<ar|en>/sounds/`: letter sounds (recorded, not used by the games yet); `audio/<ar|en>/abdullah/`.
  - `recording-sheet.csv` lists every file with its exact text.
- Each game has the same small voice player (search for `recorded voices`): `voice(key)` plays a line (`'praise'` and `'try'` pick a random take), `sayLetter`, `sayWord(word, thenKey)`, and `playSeq([...urls])` chains clips, e.g. "drive through the picture that starts with…" + the letter. Lines queue behind whatever is playing and are dropped if they would start more than about 2.5 s late, so fast events don't pile up.
- The home screen plays a welcome when it opens (or on the first tap if the browser blocks autoplay); tapping a game's title says its name.
- To replace a recording, overwrite the file with the same name. The offline cache picks it up on the next build.
- On-screen text is still in Kuwaiti dialect on purpose; only the voices are Standard Arabic.

## Website screenshots

`website-screenshots/` (next to this folder in the zip, not inside it) has 18 real in-game screenshots at 1920×1080 in Arabic for the raneenvillage.com product page: the home screen, then the menu and gameplay for each game. They're named in display order (`00-…` to `17-…`). Keep them out of the game folder so the offline cache doesn't download them.

## Editing the games

- Word lists, picture pairs, postcard facts and sky-writing strokes are plain arrays near the top of each game's script (`SPELL`, `MATCH`, `POSTCARDS`, `SKY`). Arabic copy uses Kuwaiti dialect on purpose.
- Each game has a `T = { en: {...}, ar: {...} }` object with every on-screen string.
- Abdullah's head is a flat cut-out sprite (`window.ABD.headFront` / `headBack`); the body is built in three.js.
- Keep the games self-contained. If you split files, update `sw.js`'s file list.

## Testing checklist

- iPad (Safari) and an older Android phone (Chrome): frame rate, touch buttons, landscape and portrait.
- Arabic: right-to-left layout, letters render in Baloo Bhaijaan 2, numbers in the HUD stay left-to-right.
- Offline after first visit.
- Progress survives closing the browser; the home screen's sticker book and report update after playing.

## Tested before handover (20 Oct 2026)

Every page was driven headless (Chromium, WebGL) on a 390×844 touch phone and a desktop window, in Arabic and English:
- **Star Hop:** every letter on all 3 levels collected; all island-to-island jumps completed with real joystick/jump input (moving platforms and the spring included); win → next level → last level; losing all hearts; safety net; grumble stomp and side bump; little-kids mode; home button mid-level.
- **Letter Kart:** full races in all 3 worlds; every power-up (rocket, shield vs puddle, magnet, bubble hitting the kart ahead); puddles, jump, ramps; results screen left open past a full lap; home button mid-race; sticker album; phone buttons.
- **Letter Glider:** full flights on all 3 legs with landing stars, postcards, sky-writing, warm-air lift, low-flight bounce, wrong balloons feeding the parent report; next-flight chain; menu during sky-writing; phone buttons.
- **Home screen:** opened games via `?hub`, came back with the Abdullah's World button, language carried for the visit, sticker book and grown-ups report matched the saved progress, no sideways scrolling, offline cache serves every file, nothing loads from outside the folder.
- Content check: every picture starts with the letter it's filed under, no picture is used for two different words, and shared words use the same picture in both games. English X has no picture on purpose (it's never asked).

Voice check (22 Oct 2026): played every game in Arabic and English at every difficulty while logging each clip. Every voice request used a recording (no device-voice fallbacks), no audio file was missing, and the home screen greeting loaded in both languages.

Not covered by these tests (do these on real devices): what the sound actually sounds like on speakers, real touch feel and frame rate on an older Android phone and an iPad.
