---
schema_version: 1
id: EX-0000000000B6
type: requirement
---
# Requirement with ambiguous normative language

<!-- expected finding: requirement-normative-keyword (error, blocking) — SPEC.md §6.7; only ALL-CAPS BCP-14 keywords carry normative weight -->

## Problem

A lowercase `must` inside a requirement line is ambiguous normative language.

## Requirements

- [REQ-001] The exporter must emit one bundle entry per typed artifact.
