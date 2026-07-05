---
schema_version: 1
id: EX-0000000000C1
type: decision
---
# ADR-910: Decision referencing a missing target

<!-- expected corpus finding: relationship-target-not-found (error, blocking) — SPEC.md §8.3; run `rac relationships <this dir> --validate` -->

## Status

Accepted

## Context

The Related Decisions reference below resolves to nothing in this corpus.

## Decision

Reference an absent artifact, to demonstrate the finding.

## Consequences

The corpus is not gated until the reference is fixed or removed.

## Related Decisions

- EX-0000000MSNG1
