---
schema_version: 1
id: EX-0000000000C4
type: decision
---
# ADR-913: Related Decisions pointing at a requirement

<!-- expected corpus finding: relationship-target-type-mismatch (error, blocking) — SPEC.md §8.3; the edge's range is `decision` -->

## Status

Accepted

## Context

The reference below resolves to a requirement artifact, outside the
`related_decisions` range.

## Decision

Declare a range-violating edge, to demonstrate the finding.

## Consequences

The corpus is not gated until the edge names the right section.

## Related Decisions

- EX-0000000000C5
