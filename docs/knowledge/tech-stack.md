# Tech stack

## Plugin (Java)

- **Language**: Java. Sources and bytecode target Java 17 (`build.gradle`); CI and local builds use Zulu JDK 21 (`mise.toml`).
- **Build**: Gradle wrapper (`gradle/wrapper/gradle-wrapper.properties`). Plugins: `jacoco`, `org.sonarqube`, `info.solidsoft.pitest`,
  `com.gradleup.shadow`, `net.researchgate.release`.
- **SonarQube API**: compiled against the libraries of the SonarQube distribution set by
  `sonarqubeVersion` in `build.gradle`. The build downloads the
  distribution zip and extracts it to `sonarqube-lib/` on first run.
- **Runtime mechanism**: a Java agent (`Premain-Class`) that uses Javassist to transform
  SonarQube classes in the web server and Compute Engine processes.
- **HTTP**: OkHttp (`logging-interceptor`); Jackson JSR-310 at runtime.
- **ALM clients**: GitHub, GitLab, Bitbucket (Cloud and Server), Azure DevOps (`almclient` package).

## Testing

- JUnit Jupiter, Mockito, AssertJ, WireMock.
- JaCoCo for coverage, PIT for mutation testing.

## UI

- `sonarqube-webapp`: git submodule of SonarSource's webapp (TypeScript, Nx, yarn, Node 22).
- `sonarqube-webapp-addons`: the plugin's UI additions, linked into the webapp by `setup.sh`.

## Packaging and release

- `Dockerfile` and `docker-compose.yml` build a SonarQube image with the plugin and the rebuilt
  webapp. `release.Dockerfile` builds an image from the JAR and webapp zip published on upstream GitHub releases.
- Versioning follows the SonarQube version (`gradle.properties`); releases use the Gradle Release Plugin.

## CI and tooling

- GitHub Actions: `build.yml` (snapshot build, webapp build, release, SonarCloud analysis when
  `SONAR_TOKEN` is set) and `codeql-analysis.yml`.
- mise pins Java, Node, pkl, and hk; hk runs the `commit-msg` Conventional Commits check.
- Dependabot updates Gradle and GitHub Actions dependencies.
