---
schema_version: 1
id: EX-0000000000C2
type: decision
---
# ADR-911: Live decision referencing a retired one

<!-- expected corpus finding: relationship-target-superseded (warning severity, blocking by default) — SPEC.md §7.2, §8.3 -->

## Status

Accepted

## Context

This live decision references a superseded decision through
Related Decisions, not through `supersedes`.

## Decision

Keep citing retired knowledge as if current, to demonstrate the finding.

## Consequences

An agent grounded on this reference would act on retired knowledge.

## Related Decisions

- EX-0000000000C3
