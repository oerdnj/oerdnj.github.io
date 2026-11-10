# deb.sury.org

Source of <https://deb.sury.org/>, the home page of the DEB.SURY.ORG package
repositories for Debian and Ubuntu.

It is a single page built with [Hugo](https://gohugo.io/).  Every push to
`master` is built and published to GitHub Pages by
[`.github/workflows/pages.yml`](.github/workflows/pages.yml); the repository's
Pages source has to be set to "GitHub Actions" for that.

## Editing the page

All of the text is in [`content/_index.md`](content/_index.md).

* Every `## Heading` is one box of the page and one entry of the navigation
  bar.  Where a box goes on a wide screen is decided in
  [`assets/css/site.css`](assets/css/site.css) ("Page grid").
* A sponsor with a logo: put the image into `assets/images/` and add a line to
  the list that ends with `{.logos}`.  The logos are shown in a random order on
  every visit.  Images wider than 800px are scaled down when the site is built.
* A sponsor without a logo: add a line to the list under "Sponsors".

## Preview

    hugo server

Needs Hugo 0.146 or newer: `brew install hugo`, or `apt install hugo` on Debian
testing or unstable (the 0.131 in the current Debian stable is too old).  The
published site is built with the version pinned in the workflow.

## Not part of the page

`static/.well-known/matrix/` is the Matrix delegation for `sury.org`.  It is
published together with the site and has to stay valid.
