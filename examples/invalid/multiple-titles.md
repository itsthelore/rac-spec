---
schema_version: 1
id: EX-0000000000B7
type: requirement
---
# First title

<!-- expected finding: multiple-titles (error, blocking) — SPEC.md §6.2; exactly one top-level # title -->

## Problem

An artifact carries exactly one top-level title.

## Requirements

- [REQ-001] The validator MUST reject a document with more than one top-level title.

# Second title
