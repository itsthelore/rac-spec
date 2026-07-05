---
schema_version: 1
id: EX-0000000000CA
type: decision
---
# ADR-918: Decision referencing itself

<!-- expected corpus finding: relationship-self-reference (warning severity, blocking by default) — SPEC.md §8.3 -->

## Status

Accepted

## Context

The Related Decisions reference below resolves to this very artifact.

## Decision

Reference this decision from itself, to demonstrate the finding.

## Consequences

A self-edge carries no information and is rejected.

## Related Decisions

- EX-0000000000CA
