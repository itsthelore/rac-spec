---
schema_version: 1
id: EX-0000000000B4
type: requirement
---
# Requirement with a malformed requirement-line ID

<!-- expected finding: malformed-req-id (error, blocking) — SPEC.md §6.7; [REQ-1A] is not [REQ-<digits>] -->

## Problem

A requirement line's bracket ID must match the canonical shape.

## Requirements

- [REQ-1A] The parser MUST reject this line's identifier shape.
