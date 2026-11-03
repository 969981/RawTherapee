# RawTherapee upstream auto-sync and automated releases

## Source of truth
- Official upstream: https://github.com/RawTherapee/RawTherapee, branch `dev`.
- This repository: https://github.com/969981/RawTherapee.
- Repository default branch at setup: `autoenable`. GitHub scheduled workflows MUST live on the default branch.
- The `dev` branch in this repository is fast-forwarded to the official upstream `dev`.
- `rawtherapee-autosync-state` is a machine-owned branch pointing to the last *dispatched* upstream source commit, not a verification of build or release success.

## Workflow
The `.github/workflows/sync-upstream.yml` workflow runs every 3 hours (UTC) and supports manual `workflow_dispatch`.

1. Fetch upstream `dev` and fork `dev`.
2. If needed, move fork `dev` only when a fast-forward is possible; never force-push or merge unrelated fork commits.
3. Compare the upstream SHA to `rawtherapee-autosync-state`. On the first run the state branch does not exist, so a baseline build is dispatched.
4. On a new SHA dispatch the official `appimage.yml`, `windows.yml` and `macos.yml` workflows on `dev`, then advance the state branch.

Build workflows are copied unchanged from upstream `dev` into the default branch so GitHub can register their `workflow_dispatch` events. The actual workflows execute from `dev`, which is kept aligned with the official upstream. Their build dependencies, OS/architecture matrix, packaging, and nightly GitHub Release uploads are maintained upstream.

The three build workflows are independent. A successful dispatch is **not** proof that the build or publication succeeded. Inspect the Actions tab and release page for individual results:
- https://github.com/969981/RawTherapee/actions
- https://github.com/969981/RawTherapee/releases

## Recovery / operating notes
- Manual initial run: Actions -> "Sync RawTherapee upstream and build releases" -> Run workflow (`autoenable`).
- To rerun a failed build, use the corresponding Actions workflow's "Run workflow" on `dev`, or rerun that workflow's failed jobs. The sync state deliberately suppresses duplicate automatic dispatches for a previously requested commit.
- If `dev` diverges, the sync workflow intentionally fails instead of deleting local work. Review commits and reconcile them first.
- The upstream workflows publish an unofficial rolling nightly under `nightly-github-actions`; release files are subject to individual runner and dependency failures.
- Adding the workflows to the default branch does not change the repository's default branch or overwrite any of its existing files.
- Schedules are best-effort, not guaranteed to run at an exact minute, and can be disabled by GitHub policy or repository settings.
- The workflow requires effective GitHub Actions permissions for `contents: write` and `actions: write`. Repository policy may need to allow these.
- Upstream official workflows can change without notice; their copies in `autoenable` provide workflow registration. If upstream renames a workflow file, update the copies and dispatch names here.
