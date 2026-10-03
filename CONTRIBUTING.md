# Contributing

Thanks for your interest in contributing! By participating, you agree to abide by
our [Code of Conduct](./CODE_OF_CONDUCT.md). Documentation, code, comments, and
commit messages in this repository are written in **English**.

## This is a fork

This repository is Passion Factory's fork of
[mc1arke/sonarqube-community-branch-plugin](https://github.com/mc1arke/sonarqube-community-branch-plugin).
See [UPSTREAM.md](./UPSTREAM.md) for the upstream commit the fork is based on, the
list of fork-only changes, and how upstream releases are merged.

- Changes to the plugin's behavior (features, bug fixes, SonarQube compatibility)
  belong in the [upstream project](https://github.com/mc1arke/sonarqube-community-branch-plugin).
  Please contribute them there; they reach this fork when upstream is merged.
- Fork-specific changes (tooling, repository setup, documentation of the fork) are
  welcome here.

Every edit to a file that upstream owns can cause a merge conflict on the next
sync, so keep fork-only changes in new files where possible and record them in
[UPSTREAM.md](./UPSTREAM.md).

## Getting started

```bash
git clone https://github.com/passionfactory-oss/sonarqube-community-branch-plugin.git
cd sonarqube-community-branch-plugin
git submodule update --init --recursive   # the sonarqube-webapp UI sources
mise install                              # pinned JDK (Zulu 21), Node 22, and git hooks
```

Without mise, install JDK 21 and Node 22 yourself. To build a SonarQube container with
the plugin instead, see "Building the plugin from source" in [README.md](./README.md).

## Build and test

```bash
./gradlew build     # or: mise run build
./gradlew test      # or: mise run test
```

## Commit messages

We follow [Conventional Commits](https://www.conventionalcommits.org/):
`type(scope): subject`, where `type` is one of `feat`, `fix`, `docs`, `style`,
`refactor`, `perf`, `test`, `build`, `ci`, `chore`, or `revert`. The `commit-msg`
git hook installed by `mise install` checks this; merge commits are not checked.

## Pull requests

- Keep each pull request to one logical change.
- Make sure the build and tests pass.
- Use a Conventional Commits style PR title.

## Reporting bugs

Issues are enabled on this repository. For bugs in the plugin itself, check whether
they also occur upstream and report them there too.

**Security vulnerabilities**: do not open a public issue. Follow
[SECURITY.md](./SECURITY.md).
