# htzheng.github.io

Personal academic homepage of **Haitian Zheng**, built with the
[al-folio](https://github.com/alshedivat/al-folio) Jekyll theme (v0.16.3).

## Local preview

```bash
bundle install
bundle exec jekyll serve   # -> http://localhost:4000
```

Requires Ruby 3.3.x and ImageMagick (`brew install ruby@3.3 imagemagick`).

## Deploy

Pushing to `main` triggers `.github/workflows/deploy.yml`, which builds the site
and publishes it to the **`gh-pages`** branch. In the GitHub repo, set
**Settings → Pages → Source** to the `gh-pages` branch.

## Where the content lives

| Content                                       | File(s)                                                                                      |
| --------------------------------------------- | -------------------------------------------------------------------------------------------- |
| Name / site config                            | `_config.yml`                                                                                |
| Bio, position, profile photo                  | `_pages/about.md`, `assets/img/prof_pic.jpg`                                                 |
| Contact / social links                        | `_data/socials.yml`                                                                          |
| Publications (+ thumbnails, abstracts, links) | `_bibliography/papers.bib`, `assets/img/publication_preview/`                                |
| Patents                                       | `_bibliography/patents_issued.bib`, `_bibliography/patents_pending.bib`, `_pages/patents.md` |
| CV (Education / Experience / Service)         | `_data/cv.yml`, `_pages/cv.md`, `assets/pdf/Haitian_CV.pdf`                                  |
| Venue badge colors                            | `_data/venues.yml`                                                                           |

The previous hand-written static site is preserved on the `static-site` branch.
