---
schema_version: 1
id: EX-0000000000B5
type: requirement
---
# Requirement with a duplicated requirement-line ID

<!-- expected finding: duplicate-req-id (error, blocking) — SPEC.md §6.7; REQ ids are unique per file -->

## Problem

Requirement-line identifiers must be unique within one artifact.

## Requirements

- [REQ-001] The validator MUST report duplicated requirement ids.
- [REQ-001] The report MUST anchor the finding at the first duplicated line.
