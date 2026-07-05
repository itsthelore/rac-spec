---
schema_version: 1
id: EX-0000000000C9
type: prompt
---
# A prompt declaring an edge its type does not support

<!-- expected corpus finding: relationship-edge-unsupported (warning severity, blocking by default) — SPEC.md §8.2; only decisions declare `supersedes` -->

## Status

Active

## Objective

Demonstrate an out-of-domain relationship section.

## Input

- This corpus.

## Instructions

- Declare a `## Supersedes` section on a prompt artifact.

## Output

The `relationship-edge-unsupported` finding; the section produces no edge.

## Supersedes

- EX-0000000000C1
