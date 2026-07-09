---
name: update-thumbnail
description: Set or replace the click-to-zoom thumbnail (and optional separate zoom image) for one publication or patent on this al-folio site. Use when the user provides ONE or TWO images and names a paper or patent — one image = thumbnail that zooms itself on click; two images = a small thumbnail plus a separate larger image shown on click (ask which is which).
---

# Update a publication / patent thumbnail

Set or replace the click-to-zoom thumbnail for **one** bibliography entry (a paper or a patent). Accepts **one or two** images:

- **One image** → it becomes the thumbnail; clicking it zooms that same image (medium-zoom).
- **Two images** → one is the small **thumbnail** (shown in the list), the other is the larger **zoom image** shown on click (Lightbox2, at its own aspect ratio). **Ask the user which image is which** — do not assume order.

## Inputs
- **Item** (`$ARGUMENTS` or the message): which entry — a BibTeX key (`zheng2022cmgan`), a title fragment, or a short name ("CM-GAN", "ZipIR"). If empty or it matches more than one entry, list the candidates and ask the user. Do not guess.
- **Image(s)**: the pasted/attached image(s) in the conversation. Each appears as `[Image: source: /Users/…/.claude/image-cache/…/N.ext]` — use those paths (also accept explicit file paths the user gives). If none are present, ask the user to paste one.
  - **If exactly two images are provided**, ask the user which one is the **thumbnail** and which is the **zoom image** (AskUserQuestion or a direct question).
  - If more than two are provided, ask the user to clarify which one/two to use.

## Project conventions
- Bib files: `_bibliography/papers.bib`, `_bibliography/patents_issued.bib`, `_bibliography/patents_pending.bib`.
- Images live in `assets/img/publication_preview/`. An entry references them with:
  - `preview={<file>}` — the thumbnail (always set).
  - `preview_zoom={<file>}` — optional; a separate larger image opened on click. Only set this in the two-image case.
- Filenames: short, lowercase, no author/year. Convention: zoom/full image = `<slug>.<ext>`, thumbnail = `<slug>_thumb.<ext>` (two-image case); single image = `<slug>.<ext>`. Patents use the key (e.g. `pat12602755`).
- Ruby is Homebrew `ruby@3.3` — prefix shell commands with `export PATH="/opt/homebrew/opt/ruby@3.3/bin:$PATH"`.
- Rendering is already implemented in `_layouts/bib.liquid` + `_sass/_home.scss`: every thumbnail is click-to-zoom with a magnifier-plus cue + `zoom-in` cursor. Same-image entries zoom via **medium-zoom**; entries with `preview_zoom` open that image via **Lightbox2** (which needs `images: { lightbox2: true }` in the front matter of any page the entry appears on — `_pages/publications.md` already has it; add it to `_pages/about.md` too if the entry is a `selected={true}` paper shown on the home page).

## Steps
1. **Find the entry.** `grep -rniE "<key or title words>" _bibliography/*.bib`; identify the single `@type{key, … }` block and its file. If several match, ask which. Note any existing `preview` / `preview_zoom`.
2. **Resolve the image(s).** Get the cache path(s) from the most recent `[Image: source: …]` refs (or explicit paths); confirm each exists (`ls -la`, `magick identify`). If two images, **ask which is the thumbnail and which is the zoom image** before continuing.
3. **Pick filenames.** Derive `<slug>` from the key (drop leading author+year: `zheng2022cmgan` → `cmgan`, `yu2025zipir` → `zipir`; patents → the key). One image → `<slug>.<ext>`. Two → thumbnail `<slug>_thumb.<ext>`, zoom `<slug>.<ext>`. If the entry already has a `preview`, reuse that filename for the thumbnail and overwrite in place.
4. **Process into `assets/img/publication_preview/`:**
   - **Static (jpg/png):** thumbnail → `magick "<src>" -resize '800x>' -strip -quality 85 <thumb>`; zoom image → `magick "<src>" -resize '1600x>' -strip -quality 88 <zoom>` (larger cap for the enlarged view; never upscale small images).
   - **Animated GIF:** preserve animation — `magick "<src>" -coalesce [-crop WxH+X+Y] -resize '<W>x>' +repage -layers optimize <out>`. Do NOT route a GIF through `figure.liquid`/imagemagick webp (`.gif` is already excluded from the imagemagick plugin in `_config.yml`).
   Verify each: `magick identify -format '%wx%h %b' <file>`.
5. **Wire the bib entry** (Edit): ensure `preview={<thumb file>}`. In the two-image case also add/update `preview_zoom={<zoom file>}`; in the one-image case make sure there is NO `preview_zoom`. Insert any new field right after the `@type{key,` line.
6. **Lightbox2 for `preview_zoom`:** if you set `preview_zoom` and the entry shows on a page whose front matter lacks `images.lightbox2`, add `images:\n  lightbox2: true` there (publications already has it; the home page is `_pages/about.md`, needed only for `selected={true}` papers).
7. **Rebuild the local site** (detached server has no auto-reload):
   ```
   cd /Users/hazheng/code/htzheng.github.io
   export PATH="/opt/homebrew/opt/ruby@3.3/bin:$PATH"
   pkill -f "jekyll serve" 2>/dev/null; sleep 2
   lsof -ti tcp:4000 | xargs kill -9 2>/dev/null || true
   sleep 1
   nohup bundle exec jekyll serve --watch --port 4000 --host 127.0.0.1 --detach > /tmp/jekyll-thumbnail.log 2>&1
   ```
   Wait for `Server detached`; confirm the entry renders a `preview-zoomable` wrapper with the cue, and (two-image case) a `data-lightbox` anchor pointing at the zoom file.
8. **Confirm.** Report the entry, the saved filename(s) + dimensions, and screenshot the relevant page (`http://127.0.0.1:4000/publications/` or `/patents/`). Remind the user it goes live after `git push` to `main`.

## Guardrails
- Only touch the named entry and its thumbnail/zoom files. Confirm the match before overwriting an existing image.
- Keep filenames stable when replacing (overwrite in place) so nothing else churns.
- Never modify gitignored paths (`publication_pdfs/`, `TODO.md`).
