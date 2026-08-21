# TeKillA Lunik — developer handoff

For: **Noman** — deploying this to the final domain.

Everything here is verified against the live preview build, not assumed. Where
something is unresolved it says so.

---

## 1. What this is

A single-page static site for the film **TeKillA Lunik**, built with **Astro 5**.
Confidential investor material — noindex throughout, and it needs a password (see
§6, still outstanding).

| | |
|---|---|
| Stack | Astro 5 static, Tailwind 4 via Vite plugin, self-hosted fonts |
| Output | Fully prerendered HTML — **no Node runtime needed on the server** |
| Built with | Node 22.19.0, npm 10.9.3 |
| Page weight | ~240 KB first load, excluding the teaser and PDFs |
| Live preview | <https://www.productphotoandvideo.com/Lunik/> |

The preview URL is temporary hosting belonging to the producer. It is not the
final home — that is what you are setting up.

---

## 2. Build it

```bash
npm install
npm run dev      # localhost:4321
npm run build    # → dist/
```

`dist/` is the entire deliverable. Nothing else goes on the server — not `src/`,
not `node_modules/`, and **not `assets/`** (that folder holds a 299 MB video
master and a 50 MB PDF used at build time only).

---

## 3. Deploying — read this first

Astro writes **root-absolute** asset URLs (`/_astro/…`). Served from a subfolder
without configuration, every stylesheet and image 404s and the page renders as
unstyled text. `astro.config.mjs` reads a `BASE_PATH` env var to handle this:

```bash
npm run build                      # → https://example.com/
BASE_PATH=/lunik npm run build     # → https://example.com/lunik/
```

**If the site is going to the domain root — the likely case for the real
domain — just `npm run build` with no `BASE_PATH`.** The current preview build
is `/lunik`-scoped and will not work at a root.

Two consequences worth knowing:

- The folder name must match `BASE_PATH` exactly, **case included** (Linux).
- **Renaming the folder later means rebuilding.** Moving files will not fix it.

### Then upload

| What | Where | Size |
|---|---|---|
| Everything in `dist/` except the two below | web root | ~1.4 MB |
| `media/teaser.mp4` | `media/` | ~17 MB |
| `dist/docs/TeKillA-Lunik-Creative-Deck.pdf` | `docs/` | 50.4 MB |

Use **FTP/SFTP for the two large files** — cPanel's File Manager commonly
refuses uploads near 50 MB.

### Strip before uploading

Neither is automated:

- `dist/lab/` — internal component lab, not for a client-facing site
- `dist/_headers` — Cloudflare Pages format, inert on Apache; `.htaccess`
  replaces it

---

## 4. Server configuration

`public/.htaccess` ships in the build and is **confirmed working on Apache** —
verified live: `X-Robots-Tag`, `X-Frame-Options: DENY`, `no-store`, and
`206 Partial Content` on byte-range requests (that last one is what lets viewers
scrub the teaser). It sets:

- noindex / nofollow / noarchive on every response, including the PDFs and video
- long immutable caching on fingerprinted `_astro/*` assets
- `Accept-Ranges: bytes`, and no gzip on video (gzip breaks range requests)
- correct MIME types for `.webp`, `.woff2`, `.mp4`, `.pdf`

**On nginx none of this applies** — `.htaccess` is ignored entirely and the
equivalent directives must go in the server block. If the final host is nginx,
port it rather than assuming the protections carried over.

**`robots.txt` is only honoured at the domain root.** If the site ends up in a
subfolder, add `Disallow: /<folder>/` to the *root* `robots.txt` as well.

---

## 5. Move the teaser to Vimeo — one line

The teaser is currently **self-hosted**. `media/teaser.mp4` is a web encode of
the master: 299 MB → about 17 MB, 720p H.264/AAC, with `+faststart` so it
streams instead of downloading in full first.

Two reasons it should not stay:

- the file is directly downloadable by anyone who opens devtools
  (`controlslist="nodownload"` only hides the menu item)
- every full view costs about 17 MB of egress

Upload to Vimeo as **private + domain-restricted**, then in
`src/pages/index.astro`:

```js
const VIMEO_ID = '123456789';   // currently ''
```

`TeaserPlayer` prefers `vimeoId` over `videoSrc` whenever both are set, so the
self-hosted file is ignored the moment an ID exists. Nothing else changes.
Rebuild, redeploy, and delete `media/teaser.mp4` from the server.

Either path keeps the click-to-load facade — nothing is fetched until the viewer
presses play.

> **Note on the current preview server:** the file live at
> `/Lunik/media/teaser.mp4` is the **299 MB master**, not the optimized encode —
> the wrong file was uploaded. Irrelevant once Vimeo takes over; if the
> self-hosted path persists, replace it with `dist/media/teaser.mp4`.

---

## 6. Outstanding — not code, but blocking

### There is no password gate

Anyone with the URL can read the site, download both decks and watch the teaser.
The noindex headers and `robots.txt` keep it out of **search results**; they stop
nobody who has the link.

The original design assumed Cloudflare Access (per-investor email allowlist).
That is gone with the move to shared hosting. On cPanel the replacement is
**Directory Privacy** on the site folder — a shared HTTP Basic password, which
also covers the PDFs and the mp4 since they sit under the same directory. An
`AuthType Basic` block is in `public/.htaccess`, commented out, if you would
rather configure it by hand.

**This should be in place before the link is circulated** — see the next item.

### The creative deck has an unresolved rights question

A page-by-page review of the 110-page creative deck found third-party material:
a visibly watermarked Getty/Keystone comp (p. 73), a real *TIME* cover with
masthead intact (p. 58), cast headshots of named living actors marked NON
CONTRACTUAL (pp. 20–25), and film stills throughout the aesthetics and
visual-guide sections.

The distinction: this is normal inside a private pitch PDF passed hand to hand,
and materially different once it is a downloadable file on a public URL. It is
currently hosted and downloadable.

The producers decided to host it knowingly. Flagged here so it is not a surprise
later, and because it is the reason the password is not optional. The fix, if
wanted, is a redacted deck build swapped in at the same filename — no code change.

Full detail in `README.md` §3.

### The contact address is a placeholder

The contact block renders literally `[ contact address to be supplied ]` — this
is live right now. There is no `mailto:` anywhere in the build. It was left as
plain text rather than a broken link on purpose. Set it in
`src/content/film.ts` → `TEAM`.

---

## 7. Traps that have already caught someone

- **`Compress-Archive` (Windows PowerShell) writes backslash path separators.**
  Linux extractors read `docs\file.pdf` as a filename, not a folder, so nested
  directories silently vanish on extraction. This already caused a missing
  `docs/` folder on the server. Use a tool that writes forward slashes, and
  verify with `unzip -l` before uploading.
- **Python's `http.server` ignores `Range` headers**, so video seeking appears
  broken when testing locally. Apache handles it correctly — don't chase it.
- **`_headers` does nothing outside Cloudflare Pages.** It looks like it is
  working because the file is present. It is not.

---

## 8. Where things live

```
src/pages/index.astro      the whole page; VIMEO_ID and deck links at the top
src/content/film.ts        all copy as data — timeline, credits, comparables
src/components/            TeaserPlayer, DossierEnvelope, Polaroid, PaperSheet…
src/assets/                source images; Astro emits sized webp at build
public/                    copied verbatim → .htaccess, robots.txt, docs/, media/
assets/                    working masters — NOT deployed
README.md                  design decisions, content provenance, rights review
```

Content is deliberately separated from markup: most copy changes are edits to
`src/content/film.ts`, not to templates.
