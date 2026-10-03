# Upstream tracking

This repository is Passion Factory's internal fork of
[mc1arke/sonarqube-community-branch-plugin](https://github.com/mc1arke/sonarqube-community-branch-plugin).
This page records which upstream commit the fork is based on, what the fork changes, and how to
merge new upstream releases. Update it every time you sync.

## Upstream

- Repository: `https://github.com/mc1arke/sonarqube-community-branch-plugin.git`
- Upstream default branch: `master`
- Fork branch: `main`

## Sync status

- Last synced upstream commit: `81c7ae1` (2026-06-01, "Return to SNAPSHOT version post release")
- Last upstream release included: `26.5.1` (SonarQube 26.5)
- Version on `main`: `26.6.0-SNAPSHOT`
- Last checked against upstream: 2026-10-03 (upstream `master` was still at `81c7ae1`)

## Fork-only changes

Keep this list current. During a sync these are the places where conflicts can appear.

- `README.md` — fork notice block directly below the introduction paragraph. Conflicts if
  upstream edits the lines around the introduction.
- `NOTICE` — fork copyright attribution. New file, so it does not conflict.
- `UPSTREAM.md` — this file. New file, so it does not conflict.

`LICENSE` is the verbatim LGPL-3.0 text from upstream. Do not edit it.

## Sync with upstream

1. Add the upstream remote, if it is not already configured:

   ```bash
   git remote add upstream https://github.com/mc1arke/sonarqube-community-branch-plugin.git
   ```

2. Fetch upstream and merge it into `main`. Merge instead of rebasing, because `main` is
   already pushed and rebasing would rewrite its history.

   ```bash
   git fetch upstream --tags
   git switch main
   git merge upstream/master
   ```

3. Resolve conflicts in the files listed in [Fork-only changes](#fork-only-changes).
4. Update the `sonarqube-webapp` submodule to the commit upstream points to:

   ```bash
   git submodule update --init --recursive
   ```

5. Update [Sync status](#sync-status) in this file, then push `main`.
