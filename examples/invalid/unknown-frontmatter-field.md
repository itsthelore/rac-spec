---
schema_version: 1
id: EX-0000000000B3
type: decision
owner: alice
---
# ADR-902: Decision with an unknown frontmatter field

<!-- expected finding: invalid-metadata-field (error, blocking) — SPEC.md §6.3; `owner` is outside the closed field set -->

## Status

Accepted

## Context

The frontmatter envelope's field set is closed; `owner` is not in it.

## Decision

Declare an unsupported envelope field, to demonstrate the finding.

## Consequences

The corpus is non-conformant until the field is removed.
