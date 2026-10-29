# PyRisk field manual router: agent instructions

This repository is the stable public address (and QR code) for **PyRisk: a field
manual for predicting Risk**, Joey Wilkes's interactive supervised machine
learning explainer using 5,141 simulated six-player Risk games. GitHub Pages
renders `index.html`, which redirects visitors to wherever the app is hosted.
The field manual's About page shows this router's QR code.

## Environments

| Environment | Address | Role here |
| --- | --- | --- |
| Dev | Local Liquid preview, also shared on the owner's tailnet | Render and check changes before a PR |
| Production destination | https://pyrisk-field-manual.vercel.app/ (Vercel Production) | The destination on `main` |

The field manual has Vercel Preview deployments per branch and Production on
`main`, documented in the site's own repository. The Production destination is
not live yet; its external check is expected to fail until deployment.

## Rules

- Change only `redirect_url` at the top of `index.html`; keep it an absolute
  HTTPS URL in double quotes. The QR files never change.
- Work on a feature branch or worktree and open a PR. Merge, or push to `main`,
  only with the owner's explicit approval for that specific change: `main` is
  what GitHub Pages publishes.
- Before a PR, render the preview and run `scripts/check_links.py` and
  `scripts/browser_check.js` as described in [MAINTENANCE.md](MAINTENANCE.md).
- Share any local preview on the owner's tailnet
  (`Tailscale serve --bg --http=<port> <port>`) so it can be reviewed from a
  phone.
