---
schema_version: 1
id: EX-0000000000P1
type: prompt
---
# Corpus review session prompt

## Status

Active

## Objective

Guide an agent through triaging a corpus review report worst-first.

## Input

- The corpus directory and the JSON review report.

## Instructions

- Fix tier 1 findings first, then tier 2; re-run the review after each fix.
- Never edit a retired artifact to silence a finding; update the referrer.

## Output

A corpus whose review report contains no tier 1 or tier 2 findings.

## Constraints

- Do not change enum values or relationship section names.

## Related Roadmaps

- EX-0000000000M1
