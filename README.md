# STA-3024 Statistical Analysis Tool — Pilot

A browser-based statistics tool for STA-3024 (Intermediate Statistics). This pilot is the
**independent-means vertical slice**: upload a CSV, pick a response and a 2-level group, and get
Welch's *t* test plus Hedges' *g\*ₛ* — computed in real R (via [WebR](https://docs.r-wasm.org/webr/latest/))
entirely in the browser. No install, no server, no data leaves the machine.

## Files

- `index.html` — the tool (self-contained: HTML, CSS, JS, and the embedded R program).
- `coi-serviceworker.min.js` — **you must add this** (see below). It lets WebR's fast channel work on GitHub Pages.
- `LICENSE` — PolyForm Noncommercial 1.0.0 (© 2026 Christian Bowers and Claude (Anthropic)). Free to use, modify, and share for noncommercial purposes — including teaching, research, and educational institutions. Commercial use, such as selling it or bundling it into a paid product or textbook, is not permitted.

## One-time setup

1. Create a **public** repository (GitHub Pages on a free account serves from public repos only).
2. Add `index.html` and `LICENSE`.
3. Download **`coi-serviceworker.min.js`** from
   <https://github.com/gzuidhof/coi-serviceworker> (open `coi-serviceworker.min.js`, click *Raw*,
   save it) and add it to the repository **next to `index.html`**.
4. In the repo: **Settings → Pages →** set *Source* to *Deploy from a branch*, branch `main`, folder `/ (root)`, Save.
5. Wait ~1 minute, then open the published URL (`https://<username>.github.io/<repo>/`).

On the very first visit the page reloads itself once — that is `coi-serviceworker` installing the
cross-origin-isolation headers WebR needs. Normal and expected.

## Notes

- First load downloads the R runtime (~30 MB) from the WebR CDN; it is cached afterward.
- Try it with `independent_means_-_babies.csv` (response `bwt`, group `smoke`).
- For production, hosting on Cloudflare Pages with a `_headers` file setting COOP/COEP removes the
  need for `coi-serviceworker` and the first-visit reload.
