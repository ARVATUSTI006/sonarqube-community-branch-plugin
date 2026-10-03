# Product guidelines

## Language and prose

- Write everything committed to this repository in English: code, comments, docs, issues, and
  pull requests. The repository is public.
- Follow the existing upstream style in upstream-owned files. Use sentence-case headings and
  short, direct sentences in fork-owned docs.

## Fork discipline

- Prefer new files over edits to upstream-owned files. Every edit to an upstream file is a
  potential conflict on the next upstream merge.
- When an upstream file must change, keep the edit minimal and record it in `UPSTREAM.md` under
  "Fork-only changes", with the conflict risk.
- Never edit `LICENSE`. Keep upstream copyright headers in source files intact.
- Report plugin bugs upstream as well as here; link the upstream issue or pull request.

## Code style

- Match the surrounding Java code: 4-space indentation, `final` fields, constructor injection,
  and the existing package layout (`almclient`, `ce`, `scanner`, `server`, `classloader`).
- New source files carry the same LGPL-3.0 header as their neighbors.
- Tests use JUnit Jupiter, Mockito, AssertJ, and WireMock, as the existing tests do.

## Pull request decoration output

- Decoration (comments, status checks, images) is user-facing on every analyzed repository.
  Changes to its wording or layout belong upstream, not in the fork.

## Commits

- Conventional Commits, enforced by the `commit-msg` hook (`hk.pkl`). Merge commits from
  upstream syncs are exempt.
