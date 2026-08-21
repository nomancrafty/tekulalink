# TeKillA Lunik — investor site

Private, gated one-pager for the feature film *TeKillA Lunik*. Astro 5 + Tailwind 4,
static output, built to the spec in `assets/CLAUDE.md`, `assets/brand-tokens.md`
and `assets/site-structure.md`.

```bash
npm install
npm run dev      # http://localhost:4321
npm run build    # → dist/
```

## Structure

```
src/
  components/   PaperSheet · Polaroid · Stamp · RedString · StoryboardPanel
                DossierEnvelope · TeaserPlayer · SlugBoard · AnchorNav
  content/      film.ts        all copy + facts, with provenance notes
  layouts/      Base.astro     fonts, motion gate, noindex
  pages/        index.astro    hero → teaser → story → deck → contact
                lab.astro      component lab, not linked from the site
  styles/       global.css     brand tokens, component patterns, motion
public/         .htaccess · _headers · robots.txt · favicon.svg
                docs/  both deck PDFs   ·   media/  teaser.mp4 (82.5MB)
```

Textures are generated with SVG `feTurbulence` in CSS rather than shipped as
image files — no requests, no bytes, and nothing to keep in sync.

---

## Before this can go live

Four things are missing and one is a legal call. None of them are code.

### 1. Teaser is self-hosted — temporary, move it to Vimeo

**Current state:** the teaser is served from `media/teaser.mp4`. The integrated
web encode reduces the 299 MB master to about 17 MB at 720p H.264/AAC and uses
`+faststart`, so playback can begin before the entire file downloads.

This is a stopgap for sign-off, not the delivery path. Two problems with it:

- the file is directly downloadable by anyone who opens devtools —
  `controlslist="nodownload"` only hides the menu item
- every full view costs about 17 MB of egress

**To hand over to Vimeo, this is a one-line change.** Upload as **private +
domain-restricted**, then set `VIMEO_ID` in `src/pages/index.astro`:

```js
const VIMEO_ID = '123456789';   // was ''
```

`TeaserPlayer` prefers `vimeoId` over `videoSrc` whenever both are set, so the
self-hosted file is ignored the moment an ID exists. Nothing else changes, and
`media/teaser.mp4` can then be deleted from the server. Rebuild and redeploy.

Either way the player is a click-to-load facade: nothing is fetched until an
investor presses play.

### 2. Contact address — blocking

**There is no contact email anywhere in the 110-page deck.** The contact block
currently renders `[ contact address to be supplied ]` as plain text rather than
a broken `mailto:`. Set it in `src/content/film.ts` → `TEAM`.

### 3. The deck IS now wired up — and this concern still stands

**Changed on request.** Both PDFs are now hosted and downloadable from The Deck
section: `docs/TeKillA-Lunik-Creative-Deck.pdf` (110pp, 50MB) and
`docs/TeKillA-Lunik-Investor-Deck-v2.pdf` (16pp, 170KB).

The rights review below was written when the deck was deliberately *not* hosted.
Publishing it does not resolve any of it — it makes it live. Re-read before the
link goes to anyone outside the production, and treat the password gate in §4 as
required rather than optional while the creative deck is up.

A page-by-page review of the PDF found material that should not be republished
on a hosted site, even a gated one:

| Where | What |
|---|---|
| p. 73, panel 56 | A **visibly watermarked Getty/Keystone comp** — an unlicensed preview image |
| pp. 60–108 | All 195 "visual guide" panels are reference photographs, incl. stills from other films |
| pp. 11–15 | Every polaroid in the aesthetics section is a still from another film |
| pp. 20–25 | Cast pages — headshots of named living actors, marked **NON CONTRACTUAL** |
| p. 58 | A real *TIME* magazine cover, masthead intact |
| pp. 26–33 | Miniatures / LED sections use third-party production photography |
| p. 109 | 1950s comic covers, one with a title you would not want on an investor site |

`CLAUDE.md` already rules the cast pages out. The Getty comp and the *TIME*
cover are the sharper problem: fine inside a private pitch PDF, materially
different once hosted.

**Recommended:** produce a redacted deck build and swap the file — the link and
filename can stay the same. Flag the Getty comp to the producers regardless: it
is in the deck they are already circulating.

### 4. The gate — not yet in place

**There is currently no gate.** Anyone with the URL can read the site, download
both decks and watch the teaser.

The site ships `noindex` meta tags, `X-Robots-Tag` response headers and a
blanket `robots.txt`, and all three are confirmed working in production — but
**none of that is a gate.** They keep the page out of search results; they stop
nobody who has the link.

The original plan was Cloudflare Access (per-investor email allowlist, so you
learn who actually opened it). That is gone with the move to shared hosting.
On cPanel the replacement is **Directory Privacy** on the `Lunik` folder — a
shared HTTP Basic password, which also covers the PDFs and the mp4 because they
sit under the same directory. `public/.htaccess` carries the equivalent
`AuthType Basic` block commented out if you would rather configure it by hand;
it needs an `.htpasswd` file, which the control panel creates for you.

---

## Deploying

Live reference (approval/preview): <https://www.productphotoandvideo.com/Lunik/>

It is served from a **subfolder**, which Astro does not handle by default — it
writes root-absolute asset URLs (`/_astro/…`) that 404 one level down, leaving
the page as unstyled HTML. `astro.config.mjs` reads `BASE_PATH` to fix this:

```bash
BASE_PATH=/lunik npm run build     # → example.com/lunik/
npm run build                      # → example.com/  (root deploy)
```

The folder name must match `BASE_PATH` exactly, case included. **Renaming the
folder on the server means rebuilding** — moving files alone will break it.

Then, from `dist/`:

| Upload | Where | Note |
|---|---|---|
| everything except `media/` and the 50MB deck | `lunik/` | ~1.4MB, re-upload freely |
| `media/teaser.mp4` | `lunik/media/` | optimized web encode |
| `docs/TeKillA-Lunik-Creative-Deck.pdf` | `lunik/docs/` | 50MB — use FTP if needed |

Two things to strip before handing the build over, both scripted nowhere:
delete `dist/lab/` (internal component lab) and `dist/_headers` (Cloudflare-only,
inert on Apache — `.htaccess` replaces it).

If you zip the build on Windows, **do not use `Compress-Archive`**: it writes
backslash path separators, which Linux extractors read as part of the filename,
so nested folders like `docs/` silently fail to appear. Verify with `unzip -l`.

## Content provenance

`src/content/film.ts` marks every string as either verbatim from the deck or
written for this site. Worth knowing:

- **The deck contains no logline and no English synopsis.** The story section is
  *written*, from the deck's own faux-magazine premise spread (pp. 6–8, quoted
  verbatim) and the Spanish "ESTRUCTURA DE LA HISTORIA" beat charts (pp. 17–19).
  It is not invented, but it is not lifted either — the director should read it.
- **The deck has no storyboards.** What looks like storyboards is a photo
  lookbook. The site instead uses the film's own rendered B&W frames from
  `../Storyboard/`, plus the 195 scene *headings* set as type (`SlugBoard`) —
  the headings are the film's own writing and carry no rights exposure.
- **No budget figure appears anywhere in the deck**, so the site states none.
- **The production timeline is the revised 2026–2028 schedule**, supplied by
  production. It supersedes the deck's p.4 spec sheet, which still shows the
  original 2024/25 dates — worth correcting in the deck too.
- Roles are described; no actor is named anywhere on the site.
- The deck's index (p. 9) mis-numbers its own sections; the dividers are right.
- `TEAM.designCredit` — the deck's final slide also credits "RFAEL 'RAM'
  MANDUJANO", almost certainly a typo for Rafael. Not reproduced; confirm the
  spelling before adding.

## Design notes worth keeping

Two rules in `brand-tokens.md` needed care in practice:

- **"Headlines are always `--red-stamp`"** assumes a light paper ground.
  Red-stamp on `--wood` is ~1.6:1 and on `--kraft` ~1.6:1 — both unreadable. The
  hero title, the contact heading and the deck heading therefore sit on cream
  bands, which is what the dossier metaphor wants anyway. Every text element on
  the page was measured; all 163 pass WCAG AA.
- **`--ink-faint` is a watermark weight**, not a caption weight (~3:1 on paper).
  It is used only for genuinely decorative marks.

Motion is progressive enhancement. Content's resting state is its *final* state;
the hidden start state is only applied once an inline head script confirms the
browser can animate out of it, and a 2.5s watchdog drops the gate if the
observer never fires. An investor never sees a blank page because an animation
didn't run.
