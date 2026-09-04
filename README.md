# STA-3024 Statistical Analysis Tool

A browser-based statistics tool for STA-3024 (Intermediate Statistics), a course serving psychology
and social work majors. Choose an analysis, upload a CSV, assign column roles, and read the results
two ways: R's raw console output alongside an APA 7 table ready to paste. Everything is computed in
real R via [WebR](https://docs.r-wasm.org/webr/latest/), entirely in the browser. No install, no
server, no data leaves the machine.

The procedures were selected for the designs these fields actually use. Group comparisons on scale
scores from clinical and screening instruments, pre/post and matched-pair designs, categorical
outcomes from case records and survey items, and rank-based alternatives for the skewed, bounded,
or ordinal data such instruments routinely produce.

## Analyses

**Exploratory data analysis.** Summary statistics plus histogram, boxplot, and normal Q-Q plots,
optionally split by group, with a scatterplot when a second numeric variable is supplied.

**One-sample.** *t* test for a mean; *z* test for a proportion.

**Two-sample, dependent.** Paired *t* test for a mean difference (repeated measures or matched
pairs); McNemar's test for a difference between paired proportions.

**Two-sample, independent.** Welch's *t* test with Hedges' *g\*ₛ*; Brunner-Munzel test for
stochastic superiority; *z* test for a difference between proportions.

**Chi-square and exact tests.** Goodness of fit; exact multinomial goodness of fit for small
expected counts; independence and homogeneity; Fisher's exact test.

**Regression.** Simple and multiple linear regression.

**ANOVA.** One-way Welch's ANOVA with Games-Howell pairwise comparisons.

**All pairwise differences.** Proportion differences (Newcombe); median differences (quantile
regression, τ = 0.5); group standing in the combined sample (rank-based, MCTP); head-to-head rank
comparisons (pairwise Brunner-Munzel).

## Browser requirements

Running R in the browser needs a few web platform features that older browsers either
lack or have disabled, so there is a floor.

| Browser | Minimum |
|---|---|
| Chrome | 68+ |
| Edge | 79+ (Chromium-based only) |
| Firefox | 79+ |
| Safari, macOS | 15.2+ (December 2021) |
| Safari, iOS and iPadOS | 15.2+ |
| Samsung Internet | 15+ |

In practice anything released in the last three years works. Failures are almost always
an old machine that has stopped receiving browser updates. The tool checks for these
features at startup and replaces itself with an explanation rather than failing silently,
so a student who cannot run it is told why and what to do.

**If it does not load on a laptop,** install Chrome or Firefox and open the page there.
On Windows and macOS those browsers ship their own engines, so they work even when the
built-in Safari is too old to update. This resolves nearly every desktop failure.

**If it does not load on an iPhone or iPad,** switching browsers will not help. Apple
requires every browser on iOS and iPadOS to use the same WebKit engine as Safari, so
Chrome and Firefox there are Safari with a different interface. Update the device or use
a laptop.

**On Android,** use Chrome or Firefox directly rather than opening the tool inside
another app's embedded browser, which does not support what WebR needs.

To check whether the browser is the problem, open the developer console on the tool's
page and evaluate `crossOriginIsolated`. If it returns `false`, the page is not correctly
isolated and WebR cannot start. If the browser does not recognize the identifier at all,
it is too old.

### Why the floor exists

WebR runs R across multiple threads, which requires a shared block of memory
(`SharedArrayBuffer`). Browsers disabled that feature in 2018 after Spectre and Meltdown
and re-enabled it only for pages that prove they are isolated from other sites. A page
proves this by being served with two HTTP response headers:

```
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Embedder-Policy: require-corp
```

Safari added support for those headers in 15.2, which is where the minimum above comes
from. Because these are response headers they must come from the web server and cannot
be set inside the HTML, which is why GitHub Pages deployments need `coi-serviceworker`
to supply them (see [One-time setup](#one-time-setup)).

## Files

- `index.html`: the tool (self-contained: HTML, CSS, JS, and the embedded R programs).
- `coi-serviceworker.min.js`: **you must add this** (see below). It lets WebR's fast channel work on GitHub Pages.
- `LICENSE`: PolyForm Noncommercial 1.0.0 (© 2026 Christian Bowers). Free to use, modify, and share for noncommercial purposes, including teaching, research, and educational institutions. Commercial use, such as selling it or bundling it into a paid product or textbook, is not permitted.

## AI disclosure

This tool was built with substantial assistance from Claude (Anthropic). The disclosure is worth
making explicit both because the code is offered for others to use and because students in this
course are themselves asked to audit AI-produced statistical work.

Division of labor:

- Claude drafted and refactored most of the HTML, CSS, and JavaScript, and wrote much of the
  embedded R to specification.
- The statistical design is the author's: which procedures to include, and the defaults they use.
  Those defaults are deliberate and depart from what several mainstream packages do. Welch's *t*
  and Welch's ANOVA are used unconditionally rather than gated on a variance pretest; Hedges'
  *g\*ₛ* uses a non-pooled standardizer; Games-Howell comparisons are reported without an omnibus
  gate; proportion intervals use the Newcombe hybrid-score method; familywise error is controlled
  with Holm. Reporting follows APA 7.
- Output was checked against reference results from established statistical software before
  classroom use.

Responsibility for the correctness of these analyses rests with the author, not with the model.
Users should verify results independently for any consequential application, as with any
statistical software.

The copyright notice names only the human author. Material generated by an AI system without
sufficient human authorship is not itself copyrightable, so listing the model as a copyright
holder would assert something the law does not recognize.

## One-time setup

1. Create a **public** repository (GitHub Pages on a free account serves from public repos only).
2. Add `index.html` and `LICENSE`.
3. Download **`coi-serviceworker.min.js`** from
   <https://github.com/gzuidhof/coi-serviceworker> (open `coi-serviceworker.min.js`, click *Raw*,
   save it) and add it to the repository **next to `index.html`**.
4. In the repo: **Settings → Pages →** set *Source* to *Deploy from a branch*, branch `main`, folder `/ (root)`, Save.
5. Wait ~1 minute, then open the published URL (`https://<username>.github.io/<repo>/`).

On the very first visit the page reloads itself once; that is `coi-serviceworker` installing the
cross-origin-isolation headers WebR needs. Normal and expected.

## Notes

- First load downloads the R runtime (~30 MB) from the WebR CDN; it is cached afterward.
- Some analyses install additional R packages on first use, which adds a one-time download.
- The WebR version is pinned in `index.html` rather than tracking `latest`, so the runtime
  cannot change mid-semester. WebR releases have moved R versions and broken package
  binary compatibility. Bump the pin between terms, then retest Welch's *t*, an EDA with
  plots, and a chi-square before deploying.
- Try it with `independent_means_-_babies.csv` (response `bwt`, group `smoke`).
- For production, hosting on Cloudflare Pages with a `_headers` file setting COOP/COEP removes the
  need for `coi-serviceworker` and the first-visit reload.
