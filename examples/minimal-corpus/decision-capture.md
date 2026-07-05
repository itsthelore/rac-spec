---
schema_version: 1
id: MIN-01JYX0000002
type: requirement
---
# Decision capture

## Status

Accepted

## Problem

Product decisions are invisible to the tools that need them.

## Requirements

- [REQ-001] The team MUST record each significant decision as one decision artifact.

## Success Metrics

- Every merged change that reverses a prior decision cites the ADR it supersedes.

## Risks

- The corpus goes stale if capture is not part of the review workflow.

## Related Decisions

- MIN-01JYX0000001
