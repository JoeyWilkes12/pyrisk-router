# Change, revert, and publish the PyRisk field manual router

The public address and QR code stay fixed:

```text
https://joeywilkes12.github.io/pyrisk-router/
```

Change only `redirect_url` at the top of `index.html`. GitHub Pages renders the
JavaScript redirect, HTML refresh, fallback link, canonical link, and metadata
from that one setting. The router is served by GitHub Pages; pointing it at a new
host does not deploy or change the PyRisk field manual app itself.

This initial checkout is local only: no remote, publication, or release tag has
been created. The Production destination is not live yet; its HTTP check is
expected to fail until it is deployed. The publication steps below are for
future owner-approved maintenance.

## 1. Choose where to work

The router is an **independent repository**. Commands must run inside the
router checkout.

```sh
cd /Users/joeywilkes/Desktop/scripts/pyrisk-router
git rev-parse --show-toplevel
git remote get-url origin
git status --short
```

Once the owner creates and connects the public repository, the remote must be
`https://github.com/JoeyWilkes12/pyrisk-router.git`.
Start with a clean checkout and a fresh feature branch:

```sh
git fetch origin
git switch main
git pull --ff-only origin main
git switch -c update-router-destination
```

An agent working in a git worktree should create it from the latest `main` on its
own feature branch (`git worktree add ../pyrisk-router-<task> -b <branch> origin/main`)
and run every command below from that worktree.

## 2. Change the destination

Edit `index.html`. For the Vercel Production deployment, set:

```yaml
redirect_url: "https://pyrisk-field-manual.vercel.app/"
```

Keep an absolute HTTPS URL in double quotes. The QR files do not change.

```sh
git diff --check
git diff -- index.html
```

## 3. Check before committing

Check the new destination through its HTTP redirect chain:

```sh
curl --fail --show-error --location --output /dev/null \
  --write-out 'HTTP %{http_code}: %{url_effective}\n' \
  https://pyrisk-field-manual.vercel.app/
```

Then render a local preview and run the automated checks. A raw
`python3 -m http.server` over the source directory does **not** process Liquid.
The following preview uses the Liquid engine; GitHub Pages performs the final
Jekyll build after publication.

```sh
ROUTER_GEMS="$(mktemp -d "${TMPDIR:-/tmp}/router-gems.XXXXXX")"
GEM_SPEC_CACHE="$ROUTER_GEMS/spec-cache" gem install liquid --version 4.0.4 --install-dir "$ROUTER_GEMS" --no-document
ROUTER_PREVIEW="$(mktemp -d "${TMPDIR:-/tmp}/router-preview.XXXXXX")"
ruby -I "$ROUTER_GEMS/gems/liquid-4.0.4/lib" scripts/render_preview.rb \
  --output-dir "$ROUTER_PREVIEW"
python3 -m http.server 4181 --bind 127.0.0.1 --directory "$ROUTER_PREVIEW"
```

Keep that terminal running. In a second terminal, from the same checkout:

```sh
python3 scripts/check_links.py --site-url http://127.0.0.1:4181/pyrisk-router/
```

The checker verifies the rendered destination in all references, follows the
external destination's redirects, fails on HTTP 4xx/5xx, and checks both QR
files, the sitemap, and the robots file. Local checks finish before the external
destination fetch. The repository link is fetched only in public-site mode,
because the initial local checkout has no GitHub repository yet.

For the browser regression check, use Playwright CLI:

```sh
npx --yes --package @playwright/cli playwright-cli \
  --session router-check open http://127.0.0.1:4181/start.html
npx --yes --package @playwright/cli playwright-cli \
  --session router-check run-code --filename scripts/browser_check.js
npx --yes --package @playwright/cli playwright-cli \
  --session router-check close
```

The browser test covers 390px and 1440px viewports, JavaScript enabled/disabled,
the fallback link, Back-button behavior, keyboard access, and overflow. It
substitutes a local destination to keep automated browser activity within the
preview. The HTTP checker verifies the actual external destination.

To review the rendered preview from another tailnet device, publish it on the
tailnet only while reviewing:

```sh
# Use an unused serving port; preserve mappings for other apps.
# Set TAILSCALE_BE_CLI=1 when scripting the macOS Tailscale CLI.
Tailscale serve status
Tailscale serve --bg --http=<port> <port>
Tailscale serve --http=<port> off
```

## 4. Commit, push, and open a PR

```sh
git diff --check
git add index.html
git commit -m "Update PyRisk field manual router destination"
git push -u origin HEAD
gh pr create --repo JoeyWilkes12/pyrisk-router --base main \
  --title "Update PyRisk field manual router destination" \
  --body "Updates the redirect destination. Local redirect and hyperlink checks passed."
```

If you also changed documentation or scripts, stage those specific files too.
Opening and pushing a feature branch does **not** publish the router because
Pages publishes `main`.

An agent must give you the specific PR link, summary, and validation results and
wait for your affirmative approval before merging that PR. A request to publish
does not approve a PR merge.

## 5. Publish after approval

After you approve the specific PR, merge it on GitHub. The corresponding CLI
command is below; replace `123` with the real approved PR number:

```sh
gh pr merge 123 --repo JoeyWilkes12/pyrisk-router --squash
```

Do not enable auto-merge to skip that approval. Merging updates `main`, which
starts GitHub Pages publication.

When publishing, configure the Pages source as `main`, root folder `/`, with HTTPS enabled. Keep
`.nojekyll` absent so URL substitutions run. See
[GitHub Pages publishing sources](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site).

## 6. Confirm publication

```sh
gh run list --repo JoeyWilkes12/pyrisk-router --branch main \
  --limit 5 --json databaseId,name,headSha,status,conclusion,url
```

Choose the `pages build and deployment` run whose `headSha` matches the new
`main` commit, then:

```sh
gh run watch RUN_ID --repo JoeyWilkes12/pyrisk-router --exit-status
gh api repos/JoeyWilkes12/pyrisk-router/pages \
  --jq '{status,html_url,source,https_enforced}'
```

Then run the public checks against the merged source. In the original checkout
with a clean working tree:

```sh
git switch main
git pull --ff-only origin main
python3 scripts/check_links.py
```

From a feature worktree, leave it on its branch: `main` is usually checked out
in the original checkout, so `git switch main` fails there. Synchronize the
original checkout separately, or run `python3 scripts/check_links.py` from the
worktree after `git fetch origin` and confirm `git diff origin/main -- index.html`
is empty. After a squash merge the new commit has a different SHA, so compare
the deployed `index.html` with `main`, not the feature-branch SHA.

A successful HTTP fetch of the router is HTTP 200; the automatic redirect happens
in the browser rather than as an HTTP 301/302.

## 7. Release after an approved merge

Update [CHANGELOG.md](CHANGELOG.md) for the release before merging. After the
owner approves and merges the PR, synchronize `main` and verify the changes and
checks above. Tag the release on `main` using the appropriate patch, minor, or
major version described in the README:

```sh
git switch main
git pull --ff-only origin main
git tag -a vX.Y.Z -m "Release vX.Y.Z"
```

Push the tag only with the owner's explicit approval:

```sh
git push origin vX.Y.Z
```

Do not create or push a tag during the initial local-only setup.

## 8. Restore an earlier destination

The easiest rollback is another destination change: put the old URL back in
`redirect_url`, run the checks, then follow the same commit/PR/publish process.

```sh
git log --oneline -- index.html
git show COMMIT_SHA:index.html
git restore --source=COMMIT_SHA -- index.html
git diff -- index.html
```

Published rollbacks should be new commits; avoid rewriting shared `main` history
or force-pushing. See [Git's revert documentation](https://git-scm.com/docs/git-revert).

## Command references

- [GitHub CLI: create a PR](https://cli.github.com/manual/gh_pr_create)
- [GitHub CLI: merge a PR](https://cli.github.com/manual/gh_pr_merge)
- [GitHub CLI: list workflow runs](https://cli.github.com/manual/gh_run_list)
