---
schema_version: 1
id: EX-0000000000CB
type: decision
---
# ADR-919: A typed artifact at a reserved filename

<!-- expected corpus finding: okf-reserved-filename-collision (error, blocking) — SPEC.md §5; run `rac validate <this dir>`; OKF reserves index.md and log.md for generated entry points -->

## Status

Accepted

## Context

This typed artifact is named `index.md`, which the OKF carrier reserves for
the generated bundle entry point.

## Decision

Collide with the reserved filename, to demonstrate the finding.

## Consequences

The corpus is non-conformant until the file is renamed.
