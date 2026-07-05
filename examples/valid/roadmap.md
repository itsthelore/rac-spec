---
schema_version: 1
id: EX-0000000000M1
type: roadmap
---
# Deterministic gate roadmap

## Status

Planned

## Horizon

now

## Outcomes

- A corpus whose health is checkable in CI with zero network access.

## Initiatives

- Ship the deterministic validation engine.
- Wire the gate into the pull-request pipeline.

## Success Measures

- The gate exits identically on identical corpus states.

## Assumptions

- CI runners provide no network access to the gate step.

## Risks

- Policy knobs could reintroduce nondeterminism if read from outside the corpus.

## Related Decisions

- EX-0000000000D1
