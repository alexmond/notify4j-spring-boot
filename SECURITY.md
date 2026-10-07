# Security policy

## Reporting a vulnerability

Please report security issues privately through
[GitHub security advisories](https://github.com/alexmond/notify4j-spring-boot/security/advisories/new),
not in a public issue. You should get a first answer within a week.

## Supported versions

Each Spring Boot line is a branch. Fixes go to the lines listed in the README compatibility table.

## Scope

This repo holds the Spring Boot starter only. Channel delivery, URL parsing and credential
redaction live in `notify4j-core`; see the
[notify4j security policy](https://github.com/alexmond/notify4j/blob/main/SECURITY.md).

Channel URLs under `notify4j.urls` usually contain credentials (webhook tokens, routing keys).
Keep them out of source control and supply them through your secret store or environment.
