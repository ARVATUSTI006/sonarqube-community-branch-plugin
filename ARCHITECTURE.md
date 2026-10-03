# Architecture

This document is a map of the codebase: what the plugin does, where to start reading, what each
package is responsible for, and the rules the code relies on. It describes the current state.
For how this fork relates to upstream, see [UPSTREAM.md](UPSTREAM.md).

## Bird's-eye overview

SonarQube Community Edition analyzes only one branch per project and has no pull request
analysis. Those features exist in SonarQube's code, but they are switched off unless the
commercial edition is detected. This plugin turns them on for Community Edition and supplies the
pieces Community Edition leaves out: branch and pull request parameter handling in the scanner,
branch storage on the server, pull request decoration on four ALMs (GitHub, GitLab, Bitbucket
Cloud and Server, Azure DevOps), and the matching web UI.

The decision that shapes everything else: **the plugin changes SonarQube from the inside.** A
Java agent rewrites a few SonarQube classes at JVM startup so that the server and Compute Engine
believe a branch-capable edition is running. The plugin code then registers itself as a
SonarQube *core extension*, not only as an ordinary plugin, so it can use SonarQube's internal
APIs (`org.sonar.server`, `org.sonar.ce`, `org.sonar.db`). The cost is tight coupling: each
plugin release targets exactly one SonarQube version.

```
                       one JAR: sonarqube-community-branch-plugin-<version>.jar
                                                │
        ┌───────────────────────────────────────┼───────────────────────────────────────┐
        │ Scanner (analysis on CI)              │ Web server JVM                         │ Compute Engine JVM
        │                                       │ -javaagent:...jar=web                  │ -javaagent:...jar=ce
        ▼                                       ▼                                        ▼
 CommunityBranchPluginBootstrap        CommunityBranchAgent.premain           CommunityBranchAgent.premain
   └─ elevated classloader               └─ patch edition/feature checks        └─ patch edition/feature checks
   └─ CommunityBranchPlugin.define       CommunityBranchPlugin.load (core ext.)  CommunityBranchPlugin.load (core ext.)
        │                                  └─ branch feature + support             └─ CommunityReportAnalysisComponentProvider
        │                                  └─ ALM binding / PR web services          └─ PullRequestPostAnalysisTask
        ▼                                  └─ ALM validators + client factories       └─ <ALM>PullRequestDecorator
 branch / PR configuration                                                                  └─ ALM client ──► GitHub, GitLab,
 (sonar.branch.*, sonar.pullrequest.*,                                                                       Bitbucket, Azure DevOps
  or CI auto-configuration)  ── analysis report ──────────────────────────────────────────►
```

The web UI is a separate artifact: SonarQube's own webapp (`sonarqube-webapp` submodule), rebuilt
with this repository's additions (`sonarqube-webapp-addons/`) and deployed in place of the
server's `web` directory.

## Entry points

Start with `src/main/java/com/github/mc1arke/sonarqube/plugin/CommunityBranchPlugin.java`. Its
`load` method (web server and Compute Engine) and `define` method (scanner) list every component
the plugin registers, grouped by the SonarQube process that runs it.

Then follow the process you care about:

- **Startup and the Java agent**: `CommunityBranchAgent.java` (the `Premain-Class` in
  `build.gradle`) and `CommunityBranchPluginBootstrap.java` (the `Plugin-Class`). Together they
  explain why the agent is mandatory on the server and Compute Engine.
- **Scanner side**: `scanner/CommunityBranchConfigurationLoader.java` decides whether an analysis
  is a branch or a pull request, from explicit properties or from `scanner/autoconfiguration/`.
- **Pull request decoration**: `ce/pullrequest/PullRequestPostAnalysisTask.java` runs after each
  analysis and hands off to the decorator for the project's ALM, for example
  `ce/pullrequest/github/GithubPullRequestDecorator.java`.
- **Server-side branch storage**: `server/CommunityBranchSupportDelegate.java` turns a submitted
  analysis into branch or pull request component rows.
- **Web UI**: `sonarqube-webapp-addons/src/index.ts` and `sonarqube-webapp-addons/setup.sh`.

Paths in this section and below are relative to
`src/main/java/com/github/mc1arke/sonarqube/plugin/` unless they start at the repository root.

## Module structure

### Root package

Five classes that wire the plugin into SonarQube.

- `CommunityBranchAgent` — Java agent. With argument `ce`, it makes
  `org.sonar.core.platform.PlatformEditionProvider` report the Developer edition and enables
  `org.sonar.server.almsettings.MultipleAlmFeature`. With argument `web`, it enables
  `MultipleAlmFeature` and injects a Developer edition provider into the new-code-period
  `SetAction` and `UnsetAction` web services. In both modes it flips
  `CommunityBranchPluginBootstrap.isAvailable()` to `true`. It uses Javassist to rewrite method
  bodies and constructors, then retransforms the loaded classes.
- `CommunityBranchPluginBootstrap` — the class SonarQube's plugin loader instantiates. On the
  server and Compute Engine it only checks that the agent ran, and fails startup if not. On the
  scanner it loads `CommunityBranchPlugin` through an elevated classloader and delegates to it.
- `CommunityBranchPlugin` — both a `Plugin` (scanner extensions) and a `CoreExtension` (server and
  Compute Engine extensions). It is registered as a core extension in
  `src/main/resources/META-INF/services/org.sonar.core.extension.CoreExtension`, which SonarQube
  finds because the agent JAR is on the JVM's system class path.
- `CommunityPlatformEditionProvider` — the edition provider the agent injects.

### `classloader/`

SonarQube isolates plugins from its core classes. `ElevatedClassLoaderFactory` and
`ClassReferenceElevatedClassLoaderFactory` build a classloader that uses SonarQube's core
classloader as its parent, so the scanner-side plugin code can reach classes such as
`org.sonar.scanner.*` that an ordinary plugin cannot see.

### `scanner/`

Runs inside the scanner during analysis.

- `CommunityBranchConfigurationLoader` reads `sonar.branch.name` or
  `sonar.pullrequest.key`/`branch`/`base`, rejects a mix of branch and pull request properties,
  and falls back to CI auto-configuration when neither is set.
- `BranchConfigurationFactory` builds the `CommunityBranchConfiguration`, resolving the target
  branch from the project's existing branches (`CommunityProjectBranchesLoader`).
- `autoconfiguration/` holds one `BranchAutoConfigurer` per CI system: Azure DevOps, Bitbucket
  Pipelines, Cirrus CI, Codemagic, GitHub Actions, GitLab CI, and Jenkins. Each reads that CI's
  environment variables.
- `ScannerPullRequestPropertySensor` adds GitLab CI context (project URL, pipeline ID) to the
  analysis report for the GitLab decorator.

**Key abstractions**: `BranchAutoConfigurer`, `CommunityBranchConfiguration`

### `server/`

Runs in the web server.

- `CommunityBranchFeatureExtension` advertises the `branch-support` feature.
- `CommunityBranchSupportDelegate` creates branch and pull request components when the server
  receives an analysis. It writes through SonarQube's own `DbClient` DAOs.
- `MonoRepoFeature` advertises SonarQube's `monorepo` feature as available.
- `pullrequest/ws/binding/action/` — web service actions that set, validate, and delete a
  project's ALM binding, one `Set*BindingAction` per ALM.
- `pullrequest/ws/pullrequest/` — `PullRequestWs` with list and delete actions.
- `pullrequest/ws/support/` — `SupportWs` and its info action.
- `pullrequest/validator/` — one `Validator` per ALM, used when validating a binding.

### `ce/`

Runs in the Compute Engine, which processes analysis reports.

- `CommunityReportAnalysisComponentProvider` registers the Compute Engine components:
  `PullRequestPostAnalysisTask`, the issue visitors, report and Markdown factories, the four
  decorators, and the four ALM client factories.
- `CommunityBranch` and `CommunityBranchLoaderDelegate` describe the branch being processed.
- `pullrequest/PullRequestPostAnalysisTask` runs after each analysis (a SonarQube
  `PostProjectAnalysisTask`). It does nothing unless the analysis is a pull request with an ALM
  binding, a decorator for that ALM, a revision, and a quality gate. Otherwise it builds
  `AnalysisDetails`, calls the decorator, and stores the returned pull request URL with
  SonarQube's `BranchDao`.
- `pullrequest/{github,gitlab,bitbucket,azuredevops}/` — one decorator per ALM. Each posts the
  provider's form of status: checks, commit statuses, summary comments, and inline discussions.
- `pullrequest/report/` — `ReportGenerator`, `AnalysisSummary`, `AnalysisIssueSummary`: the
  provider-neutral content of a decoration.
- `pullrequest/markup/` — a small document model (`Document`, `Heading`, `Paragraph`, `Image`, …)
  and `MarkdownFormatterFactory`, which renders it.

**Key abstractions**: `PullRequestBuildStatusDecorator`, `AnalysisDetails`, `ReportGenerator`

### `almclient/`

HTTP clients and API models for each ALM, created through factories that both the server
(binding validation) and the Compute Engine (decoration) use.

- `github/` — wraps `org.kohsuke.github` over OkHttp. `GithubClientFactory` supports GitHub App
  credentials (JWT to installation token).
- `gitlab/` — `GitlabRestClient` on Apache HttpClient with a personal access token.
- `bitbucket/` — `BitbucketCloudClient` (OAuth client credentials) and `BitbucketServerClient`
  (bearer token), both on OkHttp; models under `model/cloud` and `model/server`.
- `azuredevops/` — `AzureDevopsRestClient` on Apache HttpClient with a personal access token.

ALM credentials come from SonarQube's ALM settings and are decrypted with SonarQube's settings
encryption; the plugin stores none of its own.

### `src/main/resources/static/`

Images (badges, severity and issue-type icons, coverage and duplication charts) that decorations
link to. They are served by SonarQube by default; the "Images base URL" setting points
decorations elsewhere when the ALM cannot reach the server.

### `sonarqube-webapp-addons/`

TypeScript library `sq-server-addons` that re-implements frontend features removed from the
SonarQube Community Build webapp, mostly refactored from `SonarSource/sonarqube-webapp`
2025.1. `setup.sh` symlinks it into the `sonarqube-webapp` submodule as `libs/sq-server-addons`
for local builds. The `Dockerfile` copies it in instead, and CI builds the webapp with
`yarn nx run sq-server:build` and publishes it as `sonarqube-webapp.zip`.

### Tests (`src/test/java/`)

The test tree mirrors the main package tree. Tests use JUnit Jupiter, Mockito, and AssertJ. ALM
decorator integration tests (GitLab, Azure DevOps) stub the provider's HTTP API with WireMock and
fail on unmatched requests. `build.gradle` puts the extracted SonarQube libraries on the test
classpath and prepends a `customTestRuntime` configuration (WireMock, the GitHub API library).

## Architecture invariants

**The agent runs on both the web server and the Compute Engine.** Without it, SonarQube keeps
branch features disabled. `CommunityBranchPluginBootstrap.define` throws on either process when
the agent has not run, so a misconfigured server fails at startup rather than half-working. The
scanner never needs the agent.

**One plugin release targets one SonarQube version.** The plugin compiles against the libraries
of a single SonarQube distribution (`sonarqubeVersion` in `build.gradle`) and patches internal
classes by name. Its version's major and minor parts match that SonarQube version. Do not expect
a build to work on another SonarQube version, and do not use the manifest's `Sonar-Version`
(the minimum API version) as a compatibility signal.

**SonarQube classes are never bundled.** SonarQube libraries are `compileOnly` and
`testImplementation`. The shadow JAR contains only the plugin and its own libraries (Javassist,
OkHttp logging, Jackson JSR-310), so at runtime every `org.sonar.*` class comes from the server.

**No storage of its own.** The plugin adds no database tables, migrations, or files. Branches,
pull requests, ALM bindings, and decoration URLs are written through SonarQube's DAOs and settings.

**Decoration is pull-request-only and skips rather than guesses.** `PullRequestPostAnalysisTask`
skips branch analyses, and logs a warning and skips any pull request that lacks a pull request ID,
a binding, a decorator, a revision, or a quality gate.

**Scanner code runs only on the scanner, server code only on the server, CE code only on the
Compute Engine.** `CommunityBranchPlugin` registers each package's components for one
`SonarQubeSide`. Keep new components in the package for the process that runs them.

## Cross-cutting concerns

**Configuration**: plugin settings are SonarQube `PropertyDefinition`s registered in
`CommunityBranchPlugin` (purge of inactive branches and pull requests, branches to keep, images
base URL). Per-project ALM bindings use SonarQube's ALM settings and the binding web services.

**Logging**: SLF4J loggers, one per class. PIT is configured to ignore calls to
`org.slf4j.Logger`.

**HTTP**: each ALM client owns its HTTP stack (OkHttp or Apache HttpClient), and client factories
are injected so that tests can replace them.

**Licensing**: LGPL-3.0. Every source file carries the upstream copyright header; see `NOTICE`.

---

_Last updated: 2026-10-03. Initial version, written against SonarQube 26.5 support._

_See `docs/adr/` for Architecture Decision Records that explain why key decisions were made._
