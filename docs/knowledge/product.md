# Product guide

## What this is

Passion Factory Corp's fork of [mc1arke/sonarqube-community-branch-plugin](https://github.com/mc1arke/sonarqube-community-branch-plugin).
The plugin adds branch analysis and pull request decoration to SonarQube Community Edition.
Passion Factory uses this fork to run its in-house SonarQube Community Edition server.

## Users

- **Passion Factory engineers** whose repositories are analyzed by the in-house SonarQube server.
  They see the result as branch analyses and as pull request decoration (summary comments,
  inline issue comments, and status checks) on GitHub, GitLab, Bitbucket, or Azure DevOps.
- **Maintainers of this fork**, who merge upstream releases, keep the plugin version matched to
  the SonarQube server version, and build the plugin JAR and webapp for the in-house server.

## Goals

1. **Track upstream.** Upstream is the source of truth for plugin behavior. Merge upstream
   releases promptly and record the sync state in `UPSTREAM.md`.
2. **Keep the server upgradable.** The plugin's major and minor version must match the SonarQube
   version it runs on, so a SonarQube upgrade needs a matching plugin build.
3. **Verify release candidates.** Prove that a candidate build works in a real SonarQube
   instance (branch and pull request analysis), not only in unit tests.
4. **Keep fork-only changes small and separable.** Put them in new files where possible so that
   upstream merges stay conflict-free.

## Non-goals

- Changing plugin behavior in the fork. Behavior changes go to upstream first.
- Supporting SonarQube commercial editions or migrating data to them.
- Publishing releases or Docker images for users outside Passion Factory.

## Constraints

- **License**: LGPL-3.0, inherited from upstream. `LICENSE` stays verbatim; attribution is in `NOTICE`.
- **Upstream activity**: upstream releases have stalled at times (mc1arke/sonarqube-community-branch-plugin#1301),
  so the fork may need to carry changes, such as support for a new SonarQube version, before upstream merges them.
- **Public repository**: everything committed here is public, in English.
