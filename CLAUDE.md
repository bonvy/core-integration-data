# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project state

Freshly generated Spring Boot skeleton — the only production code is `CoreIntegrationDataApplication.kt`
and the only test is a `contextLoads()` smoke test. There is no domain logic, no schema, and no controller
yet, so most work here is greenfield.

## Commands

```bash
./gradlew build          # compile + test + package (bootJar)
./gradlew bootRun        # run the app (default port 8080)
./gradlew test           # tests only
./gradlew test --tests 'it.cashflow.core_integration_data.CoreIntegrationDataApplicationTests'
./gradlew test --tests '*ApplicationTests.contextLoads'   # single test method
./gradlew clean
```

The Gradle wrapper downloads Gradle 9.7.1 and provisions a Java 25 toolchain on first run, so the initial
build is slow. There is no linter configured.

Test reports land in `build/reports/tests/test/index.html`.

## Known blocker: no datasource

`spring-boot-starter-data-jdbc` is on the classpath but there is **no JDBC driver dependency and no
`spring.datasource.*` configuration**. As a result the Spring context fails to start:
`./gradlew build` and `./gradlew bootRun` both fail with `DataSourceBeanCreationException`
("Failed to determine a suitable driver class"). Before anything can run, either add a driver
(e.g. `runtimeOnly 'org.postgresql:postgresql'` plus datasource properties, or
`testRuntimeOnly 'com.h2database:h2'` for tests) or drop the Data JDBC starter.

## Stack and conventions

- Kotlin 2.3.21 on Java 25 toolchain, Spring Boot 4.1.1, Gradle 9.7.1, JUnit 5.
- Package root is `it.cashflow.core_integration_data` — **underscores, not hyphens**. The Gradle project
  name is `core-integration-data`; the mismatch is deliberate (see `HELP.md`) because
  `it.cashflow.core-integration-data` is not a valid package name. Keep new packages under the underscore form.
- Persistence is **Spring Data JDBC**, not JPA/Hibernate: no lazy loading, no dirty checking, aggregates
  are loaded and saved whole. Repositories extend `CrudRepository`; write SQL by hand with `@Query` where
  needed, and manage schema with plain SQL (`schema.sql` or a migration tool) since there is no `ddl-auto`.
- Web layer is Spring MVC (`spring-boot-starter-webmvc`), servlet stack, not WebFlux.
- JSON uses **Jackson 3** (`tools.jackson.module:jackson-module-kotlin`) — imports are `tools.jackson.*`,
  not `com.fasterxml.jackson.*`.
- springdoc-openapi is included: once a controller exists, docs are served at `/swagger-ui.html`
  and `/v3/api-docs`.
- Kotlin is compiled with `-Xjsr305=strict` (platform types from Java are treated as non-null when
  annotated) and `-Xannotation-default-target=param-property`, so an annotation on a constructor
  parameter applies to both the parameter and the generated property — relevant for validation and
  Jackson annotations on data classes.
- The `kotlin-spring` plugin auto-opens `@Component`/`@Configuration`/etc. classes; no need to mark them `open`.
