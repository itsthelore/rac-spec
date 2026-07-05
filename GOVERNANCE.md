# Governance

## Maintainers

Merge rights to this repository are held by:

- **Tom Ballard** ([@itsthelore](https://github.com/itsthelore)) — author of
  the reference implementation, `rac-core`.

<!-- TODO(human): second maintainer.
Name a second maintainer with independent merge rights before announcing
this repository publicly. The block below is the template:

- **NAME** (GITHUB) — AFFILIATION / relationship to the project.
-->

## Self-imposed constraint: no announcement under a bus factor of one

This repository SHOULD NOT be announced publicly while a single person holds
sole merge rights. A specification whose normative text one person can
change unilaterally is a personal document, not a contract; the constraint
stands until a second maintainer with independent judgment is named above.
Filling the `TODO(human)` block is a release-blocking item for the v0.1.0
announcement, tracked in `DISCREPANCIES.md`.

## Decision process

- **Editorial changes** — any maintainer may merge (see CONTRIBUTING.md).
- **Normative changes** — require an issue, discussion, and review by a
  maintainer who is not the PR author (once a second maintainer exists;
  until then, normative merges are held to the announcement constraint
  above).
- **Compatibility classification** — the merging maintainer classifies every
  normative change as patch/minor/major per SPEC.md §10.2 and records it in
  the CHANGELOG; disputes about classification are resolved in the issue
  before merge.
- **Deadlock** — if maintainers cannot agree, the change does not merge.
  The specification prefers stability over speed.

## Relationship to the reference implementation

`rac-core` is the reference implementation, but this repository is the
contract: where the two disagree, the disagreement is a bug to be resolved
explicitly (issue + fix on one side), never resolved silently by either
repository. Implementers should pin to tagged releases of this repository,
not to `main`.
