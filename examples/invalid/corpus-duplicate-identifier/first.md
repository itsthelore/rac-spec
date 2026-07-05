---
schema_version: 1
id: EX-0000000000C8
type: decision
---
# ADR-916: First claimant of a shared identifier

<!-- expected corpus finding: duplicate-artifact-identifier (error, blocking) — SPEC.md §6.4; canonical ids are unique corpus-wide -->

## Status

Accepted

## Context

Two artifacts in this corpus declare the same canonical id.

## Decision

Collide on purpose, to demonstrate the finding.

## Consequences

References to this id are unresolvable until one artifact is re-identified.
