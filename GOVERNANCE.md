# Governance

## Maintainer

Merge rights to this repository are held by:

- **Tom Ballard** ([@itsthelore](https://github.com/itsthelore)) — author of
  the reference implementation, `rac-core`.

The project is currently solo-maintained by choice. Additional maintainers
may be added by the existing maintainer; each addition is recorded here.

## Decision process

- **Editorial changes** — a maintainer may merge directly (see
  CONTRIBUTING.md for what counts as editorial).
- **Normative changes** — require an issue, discussion on it, and a pull
  request referencing it. While the project is solo-maintained the
  maintainer reviews and merges; once a second maintainer exists, a
  normative PR is reviewed by a maintainer who is not its author.
- **Compatibility classification** — the merging maintainer classifies every
  normative change as patch/minor/major per SPEC.md §10.2 and records it in
  the CHANGELOG; disputes about classification are resolved in the issue
  before merge.
- **Deadlock** — if maintainers cannot agree, the change does not merge. The
  specification prefers stability over speed.

## Relationship to the reference implementation

`rac-core` is the reference implementation, but this repository is the
contract: where the two disagree, the disagreement is a bug to be resolved
explicitly (issue + fix on one side), never resolved silently by either
repository. Implementers should pin to tagged releases of this repository,
not to `main`.
