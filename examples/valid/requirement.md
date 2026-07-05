---
schema_version: 1
id: EX-0000000000R1
type: requirement
---
# Offline validation

## Status

Accepted

## Problem

Teams cannot trust a corpus check that depends on network state: the same
bytes would pass on one machine and fail on another.

## Requirements

- [REQ-001] Validation MUST be a pure function of the corpus bytes and the committed configuration.
- [REQ-002] If a file cannot be parsed, then the validator SHALL report a structured finding for it.

## Success Metrics

- The same corpus state yields byte-identical findings on any machine.

## Risks

- Robustness caps must be generous enough that no real artifact is truncated.

## Assumptions

- The corpus is stored in version control.

## Related Decisions

- EX-0000000000D1
