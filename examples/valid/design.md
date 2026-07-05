---
schema_version: 1
id: EX-0000000000G1
type: design
---
# Findings report layout

## Status

Proposed

## Context

Reviewers read gate findings inline on a pull request and need the blocking
subset to be unmistakable.

## User Need

A reviewer must see, at a glance, which findings block the merge and which
are advisory.

## Design

Findings render grouped by enforcement class, blocking first, each with its
stable code, path, and one-line message.

## Constraints

- The layout must degrade to plain text for terminal output.

## Rationale

Grouping by enforcement class mirrors the exit-code contract, so the visual
hierarchy and the gate verdict can never disagree.

## Related Requirements

- EX-0000000000R1
