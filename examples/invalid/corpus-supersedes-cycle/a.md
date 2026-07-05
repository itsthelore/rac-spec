---
schema_version: 1
id: EX-0000000000C6
type: decision
---
# ADR-914: Decision A

<!-- expected corpus finding: relationship-cycle (error, blocking) — SPEC.md §8.4; A supersedes B and B supersedes A -->

## Status

Accepted

## Context

A and B each claim to supersede the other.

## Decision

Form a supersession loop, to demonstrate the finding.

## Consequences

A replacement ordering that loops is meaningless.

## Supersedes

- EX-0000000000C7
