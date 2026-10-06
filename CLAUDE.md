# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

The **Spring Boot starter** for [notify4j](https://github.com/alexmond/notify4j). Split out of the
notify4j repo in 2026-10 (history kept) so the starter can be released **per Spring Boot line**
while the Spring-free core keeps plain `MAJOR.MINOR.PATCH`.

```
notify4j-spring-boot-parent (pom)
├── notify4j-spring-boot-starter   — auto-config binding notify4j.*; email channel; Micrometer metrics
└── notify4j-sample                — runnable example (default profile only, never published)
```

`notify4j-core` is an **external released dependency**, pinned by the `notify4j.version` property
in the root POM — never a reactor sibling. Fix core bugs in the notify4j repo, release there, then
bump the pin here. A starter change that needs unreleased core code cannot be built here.

## Versioning & branches — per Spring Boot line

Follows the sibling Boot-starter standard (`spring-boot-config-json-schema`,
`spring-boot-actuator-extensions`, `gotmpl4j-spring-boot`):

- **Version = `<boot-version>.<revision>`** (e.g. `4.1.1.1` on Boot 4.1.1). The revision starts
  at **1** and resets to 1 on each new Boot patch.
- **`main` is always the latest Spring Boot line.** Older lines live on `<major>.<minor>`
  branches (currently `4.0`).
- When `main` moves to a new Boot **minor**, cut the outgoing line to its own branch first.
- A fix that applies to every line goes to each branch, one PR each.
- Update the compatibility table in `README.adoc` and `docs/modules/ROOT/pages/index.adoc` on
  every Boot line change or `notify4j.version` bump.

Use the **`boot-upgrade` skill** to survey branches against the latest Boot patch.

## Build & test

```bash
./mvnw verify -Pdefault             # what CI runs (on JDK 17, 21 and 25): tests + quality gates + the sample
./mvnw verify                       # starter only
./mvnw -pl notify4j-spring-boot-starter test -Dtest=NotificationsAutoConfigurationTest
./mvnw -pl notify4j-spring-boot-starter test -Dtest=NotificationsAutoConfigurationTest#methodName
./mvnw spring-javaformat:apply      # auto-format (tabs)
./mvnw -Pdefault -pl notify4j-sample spring-boot:run
```

Java 17 target. `verify` runs spring-javaformat, Checkstyle, PMD and the JaCoCo gate
(**80% line coverage**, `BUNDLE`). `package` skips the coverage gate.

> Run `versions:set` with `-Pdefault`, or the sample keeps a stale parent version and the next
> build breaks. The release workflow already does this.

## Architecture

- `NotificationsAutoConfiguration` is the single entry point, registered in
  `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`. It builds
  the `Notifications<E>` facade once the application supplies a `NotificationAdapter<E>` bean.
- `NotificationProperties` binds `notify4j.*` (urls, http, async, reminders, email).
- `EmailNotifier` is the one channel that is not a URL. It is active only when a
  `JavaMailSender` bean exists and `notify4j.email.to` is set; `spring-boot-starter-mail` is an
  **optional** dependency.
- `MicrometerNotificationMetrics` is wired only when a `MeterRegistry` is present;
  `micrometer-core` is also **optional**.
- The JPMS automatic module name `org.alexmond.notify4j.spring` is frozen. Do not rename the
  package.

## Releasing

Run the **`release-prep`** skill before tagging. `.github/workflows/maven_release.yml` (manual
dispatch) takes `branch` / `releaseVersion` / `nextVersion`; it sets the version, updates the
README install snippet, verifies, tags (**no `v` prefix**), deploys to Maven Central and opens a
GitHub release. Release the current line from `main`, older lines from their branch. Then run
**`update-docs-hub`**.

Repository secrets (`OSSRH_*`, `GPG_*`, `CODECOV_TOKEN`, `UNITRACK_TOKEN`) are provisioned from
the infra repo, not set by hand.

## Docs

Property and channel-URL reference lives in the notify4j docs
(<https://www.alexmond.org/notify4j/current/>). This repo's Antora component under `docs/` holds
the starter's install and compatibility page.
