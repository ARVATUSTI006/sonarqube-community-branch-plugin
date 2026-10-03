# Project workflow

## Guiding principles

1. **The plan is the source of truth**: all work is tracked in the track's `plan.md`.
2. **The tech stack is deliberate**: document tech stack changes in `tech-stack.md` before implementing them.
3. **Outcome-gated testing**: prove each task with tests and checks. TDD (red-green-refactor) is opt-in, per plan (`Testing: tdd`) or per task (`[TDD]`).
4. **High code coverage**: aim for >80% coverage of new code.
5. **Non-interactive and CI-aware**: prefer non-interactive commands.
6. **Fork discipline**: prefer new files over edits to upstream-owned files, and record every fork-only change in `UPSTREAM.md`.

## Task workflow

All tasks follow this lifecycle within `/please:implement`.

### Standard task lifecycle

1. **Select task**: choose the next available task from `plan.md`.
2. **Mark in progress**: change the task status from `[ ]` to `[~]`.
3. **Implement**: make the change the task describes.
4. **Prove it**: run the affected tests and the task's `Method:`. For TDD tasks, write the failing test first, then make it pass.
5. **Verify coverage**: run the coverage report. Target: >80% for new code.
6. **Document deviations**: if the implementation differs from the tech stack, update `tech-stack.md` first.
7. **Record fork-only changes**: if the task adds or edits files that upstream does not have, update "Fork-only changes" in `UPSTREAM.md`.
8. **Commit**: stage and commit with a Conventional Commits message.
9. **Update progress**: mark the task as completed in `## Progress` with a timestamp.

### Phase completion protocol

Run this when all tasks in a phase are complete:

1. **Verify test coverage**: list the files changed in the phase and make sure they are covered.
2. **Run the full test suite**: run all tests and debug failures (max 2 fix attempts).
3. **Manual verification plan**: write step-by-step verification instructions for the user.
4. **User confirmation**: wait for explicit user approval before continuing.
5. **Create a checkpoint**: commit with the message `chore(checkpoint): complete phase {name}`. If the stacked PR workflow is enabled (`workflow.stacked_pr.enabled=true` and `workflow.stacked_pr.tool=gh-stack` in `.please/config.yml`), `/please:implement` also runs `gh stack submit --auto` at this step.
6. **Update the plan**: mark the phase as complete in `plan.md`.

## Quality gates

Before marking any task complete:

- [ ] All tests pass
- [ ] Code coverage meets requirements (>80% for new code)
- [ ] Code follows the project style guidelines
- [ ] No compiler or static analysis errors
- [ ] No security vulnerabilities introduced
- [ ] Documentation and `UPSTREAM.md` updated if needed

## Development commands

### Setup

```bash
git submodule update --init --recursive   # sonarqube-webapp UI sources
mise install                              # Zulu JDK 21, Node 22, pkl, hk; installs the commit-msg hook
```

### Daily development

```bash
./gradlew build          # or: mise run build
docker compose up --build   # SonarQube with the plugin and rebuilt webapp (needs .env)
```

### Testing

```bash
./gradlew test                       # or: mise run test
./gradlew test jacocoTestReport      # coverage report in build/reports/jacoco
```

### Before committing

```bash
./gradlew build
```

The `commit-msg` hook checks the commit message format.

## Testing requirements

### Unit testing

- Every module has corresponding tests.
- Mock external dependencies; use WireMock for ALM HTTP APIs.
- Test both success and failure cases.

### Integration testing

- `scripts/integration-test.sh` builds the candidate plugin and webapp into the SonarQube Docker
  image, then runs real main, branch, and pull request analyses and waits for each Compute
  Engine task. Run it with `bash scripts/integration-test.sh`; it needs Docker Compose, curl,
  Python 3, and a free host port 9000. Stop any stack started from `docker-compose.yml` first:
  its fixed container names (`sonarqube`, `postgres`) make the test stack fail to start.
- The `integration-test-poc` workflow runs it on pull requests that change the plugin, webapp,
  Docker, or build files, and on manual dispatch.

## Commit guidelines

Follow Conventional Commits. See `Skill("standards:commit-convention")` for details.

### Types

- `feat`: new feature
- `fix`: bug fix
- `docs`: documentation only
- `style`: formatting changes
- `refactor`: code change without behavior change
- `perf`: performance improvement
- `test`: adding or updating tests
- `build`: build system or dependencies
- `ci`: CI configuration
- `chore`: maintenance tasks
- `revert`: revert a previous commit

## Definition of done

A task is complete when:

1. The code is implemented to specification.
2. The task's tests and `Method:` checks pass.
3. Code coverage meets project requirements.
4. The code passes all configured checks.
5. Fork-only changes are recorded in `UPSTREAM.md`.
6. Progress is updated in `plan.md`.
7. The changes are committed with a proper message.
