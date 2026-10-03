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
- Unmerged upstream PRs merged into the fork: mc1arke/sonarqube-community-branch-plugin#1296 and
  mc1arke/sonarqube-community-branch-plugin#1298, at head `a68cf75` (SonarQube 26.7, 26.8, and
  26.9 support; see [Fork-only changes](#fork-only-changes))
- Targeted SonarQube version: `26.9.0.129388` (webapp submodule at `sqcb-26.9.0`)
- Version on `main`: `26.9.0-SNAPSHOT`
- Last checked against upstream: 2026-10-03 (upstream `master` was still at `81c7ae1`)

## Fork-only changes

Keep this list current. During a sync these are the places where conflicts can appear.

- `README.md` — the two badges on the first lines point to this repository's SonarCloud project
  and `build.yml` on `main`, and a fork notice block sits directly below the introduction
  paragraph. Conflicts if upstream edits the badges or the lines around the introduction.
- `build.gradle` — test classpath split from upstream PR
  mc1arke/sonarqube-community-branch-plugin#1303 (scanner engine jars after the server libs, to fix
  `ProtobufRuntimeVersionException` in `snapshot (21)`). Applied verbatim; if upstream merges #1303
  the sync resolves cleanly, otherwise this conflicts when upstream edits the `dependencies` block.
- `.github/workflows/integration-test.yml`, `scripts/integration-test.sh` — end-to-end
  integration test (SonarQube in Docker, then main, branch, and pull request analyses) from
  upstream PR mc1arke/sonarqube-community-branch-plugin#1302, based on head `6d5f69a` with these
  fork modifications:
  - The script sets a dedicated Compose project name
    (`community-branch-plugin-integration-test`, overridable only through
    `INTEGRATION_TEST_COMPOSE_PROJECT`) and ignores an inherited `COMPOSE_PROJECT_NAME`, so `down -v`
    does not delete a developer's local volumes.
  - The script changes to the repository root first, so it runs from any directory.
  - The scanner container joins the Compose network (`<project>_sonarnet`) and reaches SonarQube
    at `http://sonarqube:9000` instead of using `--network host`, which does not reach the host
    on Docker Desktop.
  - The polling `curl` calls use `--max-time`, a failed Compute Engine status request is retried
    until the deadline, and startup success is tracked with a flag instead of re-checking the
    deadline.
  - A failed `docker compose down` in the `EXIT` trap no longer replaces the test's exit status.
  - The workflow declares `permissions: contents: read` and a `concurrency` group that cancels
    superseded runs, raises `timeout-minutes` from 15 to 30, and its path filter also lists
    `settings.gradle` and `.dockerignore`.

  New files, so they do not conflict unless upstream merges a different version of them; in that
  case take the upstream version and re-apply the fork modifications.
- SonarQube 26.7, 26.8, and 26.9 support — upstream PR mc1arke/sonarqube-community-branch-plugin#1298
  (stacked on mc1arke/sonarqube-community-branch-plugin#1296), merged at head `a68cf75` with a
  `--no-ff` merge so the 7 upstream commit SHAs are kept. It touches `build.gradle`
  (`sonarqubeVersion`, Java 17 → 21 compatibility), `gradle.properties`, `.env`, `Dockerfile`,
  `release.Dockerfile`, `.dockerignore`, `.github/workflows/build.yml`, `docker/`, the
  `sonarqube-webapp` submodule (`sqcb-26.9.0`), `sonarqube-webapp-addons/`, and
  `ScannerPullRequestPropertySensorTest.java`. Commit `8ed1bef` then changed two of these files in
  the fork:
  - `sonarqube-webapp-addons/setup.sh` resolves `ADDONS_DIR` to an absolute path before changing
    into the webapp directory, so a relative second argument no longer creates a dangling symlink.
  - `sonarqube-webapp-addons/src/feature-license/entitlements.ts` has the full LGPL license header
    that the other addon files carry.

  Review fix commit `95bcfcd` then changed a third file:
  - `docker/test-community-branch-entrypoint.sh` also asserts, after the restart run, that the
    managed plugin jar was not moved to `disabled-plugins`.

  If upstream merges both PRs with these commits, only those three files can conflict; keep the
  fork edits. If upstream squashes or rebases the PRs first, expect conflicts in every file listed
  above; take the upstream version, then re-apply the edits from `8ed1bef` and `95bcfcd`.
- `NOTICE` — fork copyright attribution. New file, so it does not conflict.
- `UPSTREAM.md` — this file. New file, so it does not conflict.
- `.gitignore` — appended `#Claude Code` and please plugin blocks at the end of the file.
  Conflicts if upstream edits the end of `.gitignore`.
- `mise.toml`, `mise.macos-x64.toml`, `mise.lock`, `mise.macos-x64.lock`, `.miserc.toml` — tool
  versions (Zulu 21, Node 22, hk) and tasks. New files, so they do not conflict.
- `hk.pkl` — git hook configuration (`commit-msg` Conventional Commits check). New file, so it
  does not conflict.
- `.worktreeinclude`, `.claude/settings.json` — worktree and Claude Code setup. New files, so
  they do not conflict.
- `SECURITY.md`, `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md` — community health files. New files, so
  they do not conflict.
- `ARCHITECTURE.md` — codebase map. New file, so it does not conflict; update it when an upstream
  sync changes the module structure.
- `CLAUDE.md`, `.please/`, `docs/` — Claude Code instructions and the please workspace (knowledge
  files, tracks, specs, ADRs). New files, so they do not conflict.
- `.gemini/config.yaml` — Gemini Code Assist PR review settings (draft PRs not reviewed). New
  file, so it does not conflict.

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
