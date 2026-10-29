# PyRisk router

A permanent address for **PyRisk: a field manual for predicting Risk**, an
interactive explainer by Joey Wilkes. It teaches supervised machine learning
through 5,141 simulated six-player Risk games: game-grouped splits, calibration
and uncertainty, TreeSHAP attribution, and survival analysis. The router
redirects immediately to wherever the app is hosted, so links and QR codes keep
working when the host changes.

- Public address: https://joeywilkes12.github.io/pyrisk-router/
- Current destination: https://pyrisk-field-manual.vercel.app/ (Vercel Production)
- Repository: https://github.com/JoeyWilkes12/pyrisk-router
- QR assets: `pyrisk-router-qr.png` (1176 × 1176 pixels, 300 DPI, black on
  white, high error correction) and `pyrisk-router-qr.svg`. Both encode the
  router address, not the destination.

Environment role: routes to the **Production** PyRisk field manual deployment on
Vercel. Agents: see [AGENTS.md](AGENTS.md).

Built from the owner's existing router
template. See the [maintenance guide](MAINTENANCE.md) for changing or reverting
the destination, validation, commit/push commands, PR approval, and publication.

The Production destination is not live yet. Its external destination check is
expected to fail until it is deployed. The GitHub repository and Pages address
are planned; this checkout has no remote and has not been published.

## Who links here

Link to the router address, not the host, anywhere the link should survive a
host change:

- The field manual's About page, which shows this router's QR code.
- Printed or shared QR codes.

Changing the destination here updates every one of those without republishing
the field manual.

## Change the destination

Edit only the `redirect_url` value at the top of `index.html`:

```yaml
redirect_url: "https://pyrisk-field-manual.vercel.app/"
```

Commit and push a feature branch, then open a PR. After you approve and merge it
into `main`, GitHub Pages runs Jekyll automatically and replaces every
`page.redirect_url` reference with that value. This updates the JavaScript
redirect, HTML refresh, fallback link, canonical link, and structured metadata
together. Keep the URL in double quotes and use an absolute HTTPS address.

JavaScript uses `location.replace()` so the browser's Back button doesn't loop
through the router. The HTML refresh also works with JavaScript disabled. The
visible link provides a manual way to continue if automatic navigation is blocked.
This is a browser redirect served with HTTP 200, rather than an HTTP 301/302.

Keep the repository name and Pages address stable. Allow GitHub Pages a few
minutes to publish a push.

## Publishing and checks

GitHub Pages publishes the root of the `main` branch with its built-in Jekyll build.
Do not add `.nojekyll`: it would disable the URL substitutions. Opening the source
HTML directly also skips Jekyll; preview the generated site instead.

Run the automated hyperlink and generated-content regression check before a
release, supplying a rendered local preview URL, then verify the public deployment.
The maintenance guide includes a complete preview setup using
`scripts/render_preview.rb`:

```sh
python3 scripts/check_links.py --site-url http://127.0.0.1:4181/pyrisk-router/
python3 scripts/check_links.py
```

The checker follows HTTP redirects, fails on HTTP 4xx/5xx, and checks the published
redirect destinations, metadata, sitemap, robots file, and both QR files. Browser
regression coverage checks immediate navigation with and without JavaScript,
Back-button behavior, the fallback link, keyboard focus, and overflow at 390px
and 1440px. With Playwright CLI open on that localhost preview, run
`run-code --filename scripts/browser_check.js`. The test substitutes a local
destination so the browser does not visit external sites.

`robots.txt` allows public content. Because this is a GitHub project site, its file
is under the project path; domain-wide crawler rules belong to the account's root
site. The canonical URL points crawlers to the current destination.

## QR source

Regenerate the QR files with the QR Code Generator skill only if the stable
Pages address changes:

```sh
python3 ~/.claude/skills/qr-code-generator/scripts/generate_qr.py \
  "https://joeywilkes12.github.io/pyrisk-router/" \
  --name pyrisk-router-qr --formats png,svg --box-size 24 --output-dir .
```

The four-module white border is the required quiet zone; preserve it when
printing or placing the image.

## Versioning

[CHANGELOG.md](CHANGELOG.md) follows Keep a Changelog 1.1.0 and Semantic
Versioning 2.0.0. Patch releases fix copy, metadata, or documentation; minor
releases change the destination or add checks. Major releases change the public
address or QR code, which should never happen.

After an approved merge, follow the maintenance guide to tag `vX.Y.Z` on `main`.
Push a release tag only with the owner's approval.
