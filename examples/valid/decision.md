---
schema_version: 1
id: EX-0000000000D1
type: decision
---
# ADR-001: Validation is deterministic and offline

## Status

Accepted

## Category

Architecture

## Context

A corpus gate that consults the network or a clock cannot give the same
verdict twice, so its findings cannot be required in CI.

## Decision

Every validation rule is a pure function of the corpus bytes and the
committed repository configuration.

## Consequences

Findings are reproducible and CI-enforceable; rules that would need external
state (ticket existence, link liveness) are out of scope for the engine.

## Alternatives Considered

- Best-effort online checks were rejected: they turn a merge gate into a
  flake source.

## Related Requirements

- EX-0000000000R1
