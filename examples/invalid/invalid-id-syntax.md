---
schema_version: 1
id: EX-BADID
type: decision
---
# ADR-903: Decision with a malformed artifact ID

<!-- expected finding: invalid-id-syntax (error, blocking) — SPEC.md §6.4; the suffix must be 12 Crockford base32 characters -->

## Status

Accepted

## Context

`EX-BADID` does not match the ID grammar.

## Decision

Declare a malformed identifier, to demonstrate the finding.

## Consequences

The corpus is non-conformant until the identifier is regenerated.
