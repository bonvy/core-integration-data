# core-integration-data

Spring Boot service (`it.cashflow` / `core-integration-data`) built with Kotlin and Spring Data JDBC.

## Requirements

- **JDK 25** — the Gradle build declares a Java 25 toolchain and will try to provision or locate one.
- Nothing else: Gradle itself comes from the wrapper (`./gradlew`), which downloads Gradle 9.7.1 on
  first use. The first build is therefore slow; later builds are fast.

## Commands

All commands run from the repository root. On Windows use `gradlew.bat` instead of `./gradlew`.

### Install dependencies

Dependencies are resolved automatically by Gradle on the first build. To pre-fetch them:

```bash
./gradlew dependencies
```

### Compile

```bash
./gradlew classes          # main sources only
./gradlew assemble         # compile + package the executable jar into build/libs/
```

### Test

```bash
./gradlew test             # run all tests
./gradlew test --tests '*ApplicationTests'                # a single test class
./gradlew test --tests '*ApplicationTests.contextLoads'   # a single test method
```

The HTML report is written to `build/reports/tests/test/index.html`.

### Build (compile + test + package)

```bash
./gradlew build
```

### Run

```bash
./gradlew bootRun
```

The service starts on http://localhost:8080. Alternatively run the packaged jar:

```bash
./gradlew bootJar
java -jar build/libs/core-integration-data-0.0.1-SNAPSHOT.jar
```

### Clean

```bash
./gradlew clean
```

## Current status

The project is a skeleton: it contains the application entry point and a context-load smoke test, no
domain code yet.

⚠️ **The application does not start as-is.** `spring-boot-starter-data-jdbc` is on the classpath but no
JDBC driver or `spring.datasource.*` configuration has been added, so both `./gradlew bootRun` and
`./gradlew build` fail with:

```
Failed to determine a suitable driver class (DataSourceBeanCreationException)
```

To get it running, add a database driver and datasource settings — for example in `build.gradle`:

```groovy
runtimeOnly 'org.postgresql:postgresql'
```

and in `src/main/resources/application.properties`:

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/core_integration_data
spring.datasource.username=...
spring.datasource.password=...
```

(or add `testRuntimeOnly 'com.h2database:h2'` for an in-memory database in tests).

## API documentation

springdoc-openapi is on the classpath. Once endpoints exist and the app boots, the docs are available at:

- Swagger UI: http://localhost:8080/swagger-ui.html
- OpenAPI spec: http://localhost:8080/v3/api-docs

## Tech stack

| | |
|---|---|
| Language | Kotlin 2.3.21 (Java 25 toolchain) |
| Framework | Spring Boot 4.1.1 (Spring MVC, servlet stack) |
| Persistence | Spring Data JDBC |
| JSON | Jackson 3 (`tools.jackson`) |
| API docs | springdoc-openapi 3.1.0 |
| Build | Gradle 9.7.1 (wrapper) |
| Tests | JUnit 5 + `kotlin-test-junit5` |
