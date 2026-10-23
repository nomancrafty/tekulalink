# Lunik site — update for the server

**22 October 2026 · update to a live site, not a first deploy.**

The site is already live at <https://www.productphotoandvideo.com/Lunik/>.
This release adds **Russian and Spanish**. Nothing else changed.

`HANDOFF.md` in this folder is the full technical reference — build commands,
server config, the Vimeo migration, outstanding items. Read it if you are going
to work on the project. For just applying this update, this page is enough.

---

## What to do

Upload **the contents of `upload-to-Lunik/`** into the existing `Lunik` folder
on the server, overwriting what is there.

Not the folder itself — its contents. You should end up with
`Lunik/index.html`, not `Lunik/upload-to-Lunik/index.html`.

```
Lunik/
  index.html        ← replaced (English)
  ru/index.html     ← NEW
  es/index.html     ← NEW
  _astro/           ← replaced (css, fonts, images)
  docs/             ← unchanged, investor deck included for completeness
  favicon.svg  robots.txt  .htaccess
  media/            ← LEAVE ALONE
```

## Do NOT re-upload these

They are already on the server, unchanged by this release, and total ~133 MB:

- `media/teaser.mp4` (82.5 MB)
- `docs/TeKillA-Lunik-Creative-Deck.pdf` (50.4 MB)

If your FTP client is set to sync or mirror the folder, make sure it will not
delete `media/` — it is not in this package.

## Then check it worked

Open all three and confirm each loads **styled, with images**:

- <https://www.productphotoandvideo.com/Lunik/> — English
- <https://www.productphotoandvideo.com/Lunik/ru/> — Russian
- <https://www.productphotoandvideo.com/Lunik/es/> — Spanish

Then click **EN / RU / ES** in the top right of any of them.

If a page loads as plain unstyled text with no images, the asset paths are
wrong — see "Deploying" in `HANDOFF.md`. Do not try to fix it by moving files;
it needs a rebuild. The folder must stay named exactly `Lunik`, capital L.

## If you need to roll back

Keep a copy of the current `Lunik/index.html` and `Lunik/_astro/` before
overwriting. Restoring those two reverts this release; `ru/` and `es/` can then
be deleted.

---

## Two things already on the list

Neither is caused by this update, and both are covered in `HANDOFF.md`:

1. **There is no password on the site.** It is confidential investor material
   and anyone with the URL can read it, including both deck PDFs. cPanel →
   Directory Privacy on the `Lunik` folder is the fix.
2. **The teaser is self-hosted and should move to Vimeo.** It is a one-line
   change in the source (`VIMEO_ID`), then a rebuild.
