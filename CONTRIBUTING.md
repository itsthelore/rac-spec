# Contributing to the RAC (Requirements as Code) specification

This repository holds a specification, so its change process is stricter
than a code repository's: the text is a contract other implementations build
against, and a merged change can obligate every implementation.

## Normative changes

A **normative** change is anything that alters what a conformant corpus,
producer, or consumer must do: edits to MUST/SHOULD/MAY statements in
SPEC.md, to the JSON Schemas in `schema/`, to the closed enums in
`vocabulary/` (status values, relationship types, artifact types), to the
check table in SPEC.md §9.3, or to the conformance definitions.

Normative changes follow this path, in order:

1. **Issue** describing the problem — what real corpus, implementation, or
   ambiguity motivates the change. A proposal without a motivating problem
   is closed.
2. **Discussion** on the issue until the shape of the change is agreed.
3. **Pull request** referencing the issue. The PR must update, together and
   consistently: the SPEC.md prose, the affected schema or vocabulary file,
   the examples (a new rule needs a valid/invalid example pair and a
   `manifest.json` entry), and the CHANGELOG.
4. **Maintainer review** (see GOVERNANCE.md). A normative PR is not merged
   by its author without a second maintainer's review once the project has
   one.
5. **Merge + CHANGELOG entry**, classified per the compatibility policy in
   SPEC.md §10.2 (minor: additive enum values or advisory checks; major:
   removals, semantic changes, tier reassignments, new blocking checks;
   patch: editorial).

This mirrors the ADR discipline used in the reference implementation
([`rac-core`](https://github.com/itsthelore/rac-core), `rac/decisions/`):
decisions are recorded, reviewed by someone other than the author, and never
silently overridden. Where a spec change reflects a decision recorded there,
cite the ADR in the PR.

## Editorial changes

Typos, formatting, clarified wording that does not change meaning, and new
*illustrative* examples can go straight to a pull request — no issue
required. If a reviewer judges an "editorial" change normative, it is
rerouted through the process above.

## Keeping prose and machine artifacts in lockstep

SPEC.md's normative statements are traced to `extraction-inventory.json`
(HTML comments in the source). The prose, the schemas, and the check table
must never disagree; a PR that changes one without the others is incomplete.
Where the specification and the reference implementation are found to
disagree, do not silently follow either — that is a bug in one of them: open
an issue and record the resolution.

## What will not be accepted

- New values in closed enums without the compatibility-policy dance
  (SPEC.md §10.2) — extension is via new keys, never new enum values.
- Normative statements that no validator check enforces or could enforce
  deterministically and offline.
- Describing this specification as "a standard." That status is earned by
  independent implementations, not claimed in text.
