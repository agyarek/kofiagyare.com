# AV Ecosystem Map migration runbook

> **STATUS: COMPLETED 2026-07-23.** `agyarek/av-ecosystem-map` exists and
> serves the map at https://agyarek.github.io/av-ecosystem-map/ via GitHub
> Pages (deploy from branch: `main`, root — enabled manually by the owner; the
> Actions-based enablement in step 3 turned out to be blocked for the workflow
> token, so no deploy workflow is used). The old URL redirects: the flip in
> step 4 was pushed to `claude/ecstatic-newton-intteb` as planned. The steps
> below are kept for reference only. This folder can be deleted whenever this
> branch is merged.

Goal: move the AV ecosystem map from `https://agyarek.github.io/kofiagyare.com/`
to `https://agyarek.github.io/av-ecosystem-map/`, leaving this repo to the
personal-site CMS.

## Current state (2026-07-23)

- The live map is served by GitHub Pages from branch `claude/ecstatic-newton-intteb`
  of `agyarek/kofiagyare.com` (Pages source = that branch, root folder).
- Map source of truth on that branch: `index.html`, `data/av-companies.json`,
  `data/av-enrichment.json`, `data/av-funding-timeline.json`,
  `data/av-companies.csv`, `tools/make-csv.py`, `LICENSE`.
- The target repo `agyarek/av-ecosystem-map` did not exist and could not be
  created from the Claude session (integration token lacks repo-creation
  rights). **Blocked on: the owner creating the repo.**

## Remaining steps

1. **Owner (manual, one time):** create a new **public** repo
   `agyarek/av-ecosystem-map` (empty is fine — no README/license init needed)
   and make sure the Claude GitHub app has access to it
   (github.com → Settings → Applications → Claude → Repository access), if the
   installation is limited to selected repositories.

2. **Populate the new repo** (Claude or owner). From a checkout of
   `agyarek/kofiagyare.com`:

   ```bash
   git fetch origin claude/ecstatic-newton-intteb claude/av-ecosystem-map-migration-oiyiyx
   mkdir -p /tmp/avmap && cd /tmp/avmap && git init -b main .
   cd /path/to/kofiagyare.com
   git archive origin/claude/ecstatic-newton-intteb \
       index.html data/av-companies.json data/av-enrichment.json \
       data/av-funding-timeline.json data/av-companies.csv \
       tools/make-csv.py LICENSE | tar -x -C /tmp/avmap
   # README and Pages deploy workflow prepared in this folder:
   git show origin/claude/av-ecosystem-map-migration-oiyiyx:migration/new-repo/README.md \
       > /tmp/avmap/README.md
   mkdir -p /tmp/avmap/.github/workflows
   git show origin/claude/av-ecosystem-map-migration-oiyiyx:migration/new-repo/.github/workflows/deploy.yml \
       > /tmp/avmap/.github/workflows/deploy.yml
   cd /tmp/avmap && git add -A && git commit -m "AV ecosystem map — moved from kofiagyare.com" \
       && git push -u <av-ecosystem-map-remote> main
   ```

3. **Verify the new home.** The `Deploy to GitHub Pages` workflow runs on push;
   `actions/configure-pages` with `enablement: true` creates the Pages site on
   first run. Confirm the run succeeds and
   `https://agyarek.github.io/av-ecosystem-map/` loads the map.
   (If enablement is rejected, flip it manually once: repo Settings → Pages →
   Source: **GitHub Actions**, then re-run the workflow.)

4. **Flip the old URL to a redirect — only after step 3 is verified.** Copy the
   redirect pages from this branch onto the Pages-serving branch:

   ```bash
   git checkout claude/ecstatic-newton-intteb
   git checkout origin/claude/av-ecosystem-map-migration-oiyiyx -- index.html 404.html
   git commit -m "Redirect old AV map URL to av-ecosystem-map" && git push
   ```

   `index.html` forwards the root URL (keeping query + hash); `404.html`
   forwards deep links, remapping `/kofiagyare.com/<path>` →
   `/av-ecosystem-map/<path>`.

5. **Clean up.** Delete this `migration/` folder from the migration branch
   before merging it to `main`. Optionally, later: switch the old repo's Pages
   source to `main` (Settings → Pages) so the CMS repo owns its own Pages
   config, or disable Pages there entirely once traffic to the old URL dies
   off — the redirect keeps working either way as long as Pages stays enabled.
