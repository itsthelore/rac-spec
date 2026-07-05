---
schema_version: 1
id: EX-0000000000B2
type: decision
---
# ADR-901: Decision with an unknown status value

<!-- expected finding: invalid-decision-status (error, blocking) — SPEC.md §7 -->

## Status

Retired

## Context

`Retired` is not in the decision status enum (Proposed, Accepted,
Superseded, Deprecated).

## Decision

Declare an out-of-vocabulary status, to demonstrate the finding.

## Consequences

The corpus is non-conformant until the value is corrected.
