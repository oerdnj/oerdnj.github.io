# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

The source of <https://deb.sury.org/>: a single-page Hugo site (the home page plus a 404 page), and the Matrix delegation files for `sury.org` that are published along with it. There are no tests, no linter and no package manager.

## Commands

```sh
hugo server   # live preview on http://localhost:1313/
hugo --gc     # build into public/, exactly as the deploy workflow does
```

- A build is the only check there is. It fails on template errors and on a logo that is referenced in the Markdown but missing from `assets/`.
- Hugo 0.146 or newer is required (`min` in `hugo.toml`; older versions look for templates elsewhere). The standard edition is enough, there is no Sass. The deployed site is built with the version pinned in `.github/workflows/pages.yml` — `HUGO_VERSION` and `HUGO_SHA256` have to be changed together.
- HTML minification is switched on in `hugo.toml`, so `hugo` and `hugo --minify` produce the same output.

## Deployment and hosting

- **A push to `master` is a production deploy.** `.github/workflows/pages.yml` builds with Hugo and deploys to GitHub Pages. There is no staging.
- The repository's Pages source must be "GitHub Actions". `_config.yml` is *not* configuration: it is a tripwire that makes GitHub's built-in Jekyll build fail if the source is ever "Deploy from a branch", because Jekyll would otherwise publish the README as the home page and drop the Matrix files. Leave it in place.
- Visitors do not reach GitHub directly. `deb.sury.org` and `sury.org` are both served by a CDN (a BunnyCDN pull zone) whose origin is `oerdnj.github.io`. Consequences:
  - There is deliberately no `CNAME` file and no custom domain configured in Pages (the history shows it being added and removed).
  - `baseURL` in `hugo.toml` is `https://deb.sury.org/`. Do not pass the Pages URL as `--baseURL` in the workflow (the usual Hugo recipe does): the canonical and Open Graph URLs would become `oerdnj.github.io`.
  - Asset URLs in the templates are root-relative so they work under all three host names.
  - The stylesheet URL is fingerprinted so a page cached by the CDN can never be paired with a different stylesheet. It has no `integrity` attribute on purpose, since a CDN that re-minifies assets would make browsers reject the file.
  - If a change does not show up, compare the origin with the CDN: `curl -sI https://oerdnj.github.io/` and `curl -sI https://deb.sury.org/`.
- `static/.well-known/matrix/{client,server}` delegate the Matrix server name `sury.org` to `matrix.sury.org`. They have nothing to do with the page, but they are live infrastructure: federation and client logins break if they vanish or stop being valid JSON. They have no file extension, so GitHub serves them as `application/octet-stream`; the content type is corrected at the CDN, not here. The workflow protects them three ways, all of which must survive any rewrite of it: `include-hidden-files: true` (the upload action drops dot-directories by default), a `jq` check before the upload, and a `curl` check after the deployment.

## How the page is put together

- `content/_index.md` holds all of the text, as ordinary Markdown. The one exception is the grid in "Supported Releases", which the `{{< releases >}}` shortcode draws from `data/releases.toml`.
- `layouts/home.html` splits the rendered HTML at every `<h2` and wraps each piece in `<section class="section section-ID">`, where ID is the heading's anchor (`## Bug Reporting` gives `section-bug-reporting`). The navigation bar is generated from the same headings.
- `assets/css/site.css` places those sections by class in a 12-column grid on wide screens (the "Page grid" part). A section it does not know spans the full width, so a new or renamed heading degrades gracefully instead of breaking; narrow screens are a single column in Markdown order.

  This means layout is coupled to heading text. Renaming a heading changes its anchor and its class. The anchors are also linked from outside the site, so keep them stable; if a heading has to be reworded, pin the old anchor with `## New title {#old-id}`.
- Conventions inside the Markdown that the stylesheet relies on:
  - `{.logos}` after a list of linked images makes the sponsor logo wall. The inline script at the end of `layouts/home.html`, the only JavaScript on the site, shuffles it on every visit so no sponsor is always first.
  - `{.cta}` after a paragraph turns its link into a button.
  - In "Packages", each `####` heading followed by a list is one repository card. The cards are laid out side by side purely in CSS (dense grid placement), which is why there are no wrapper elements.
  - In "Support", the single list is two levels deep on purpose: the top-level items are group labels ("Commercial support:", "Ways how to donate:") and the options are nested under them. Being one element, the list can be placed beside the text as a whole, so a label can never be separated from its options. Do not turn the labels back into paragraphs, and do not rely on CSS multi-column layout with `break-after: avoid` to keep a label with its list: Chrome honours it, but the maintainer's browser split them.
  - The "Support the project" button in the header links to `/#support`.
- The "Supported Releases" grid (`layouts/_shortcodes/releases.html`, data in `data/releases.toml`) has PHP versions down and releases across. It is maintained by hand on purpose and nothing in it depends on the date: a release with an `until` line in the data shows that date in every row; one without it says "PHP LTS" (a link), or a dash where Freexian does not offer that PHP version on that release. When a release leaves the main repository, delete its `until` line. Do not add date-based switching, in the template or in JavaScript: the maintainer tried it and chose manual upkeep, since the data has to be edited for new versions anyway.
- The grid's two colours (green for packages.sury.org, blue for PHP LTS; the maintainer did not want red there, as it reads as "no") were checked as a pair for colour-blind separation on the light and the dark background; every cell also carries text and a `title`, so colour is never the only cue. Do not swap in other colours without re-checking them.
- The inline script at the end of `layouts/home.html` is the only JavaScript on the site and does one job, shuffling the logos. The built HTML does not depend on the build date.
- Check layout changes in more than one browser engine; that Support bug passed a Chrome-only check.
- `layouts/_markup/render-image.html` handles every Markdown image: it loads the file from `assets/` (so `/images/x.png` in the Markdown is `assets/images/x.png` on disk), adds width and height, and scales anything wider than 800px down at build time. Oversized originals can therefore be committed as they come; do not resize them by hand.
- Go's HTML templates refuse to build tag names or attributes from variables. That is why `layouts/home.html` assembles the `<h2` with `printf ... | safeHTML` and `layouts/baseof.html` writes the `<h1>` and `<p>` variants of the brand out separately.

## Content

- The prose is the maintainer's own voice; do not copy-edit it unless asked.
- There are two sponsor lists with different meanings: "Acknowledgements" (companies shown with a logo) and "Sponsors" (a plain list of names). Most commits to this repository add or remove an entry in one of them.
- Some alt texts and taglines are worded at the sponsor's request (for example TouchWeb's); keep them verbatim.
- The dates in `data/releases.toml` are looked up, never estimated. For a release still in the main repository, `until` follows the policy stated on the page and Debian's `distro-info-data` (<https://salsa.debian.org/debian/distro-info-data>): `eol-lts` in `debian.csv` for Debian, `eol` (the end of Standard Support) in `ubuntu.csv` for Ubuntu LTS.
- PHP LTS by Freexian gets no end dates on the page, by the maintainer's decision: everything there is supported until Freexian says otherwise. The `not_in_lts` lists follow the availability table on <https://www.freexian.com/lts/php/>, but the maintainer's word wins where that table lags: PHP 8.6 was entered as available in November 2026, before Freexian's page listed it.
- The `flattr:id` meta tag in `layouts/baseof.html` has to stay.
