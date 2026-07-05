# Changelog

Changes to the RAC (Requirements as Code) specification, by version. The
specification versions independently of any implementation, on semantic
versioning; the compatibility rules are in SPEC.md §10.2.

## v0.1.0 — 2026-07-05

Initial extraction of the specification from the reference implementation,
[`rac-core`](https://github.com/itsthelore/rac-core).

- **SPEC.md** — the normative specification: artifact model (five types,
  frontmatter envelope, ID grammar and path-independent identity, per-type
  sections, requirement-line grammar), the closed `status` lifecycle with
  supersession semantics, the closed typed-relationship vocabulary with
  referential integrity and graph-shape rules, the finding/severity/
  enforcement model with the normative check table, conformance levels, the
  versioning and compatibility policy, and the OKF composition contract.
- **schema/** — machine-readable JSON Schemas for the frontmatter envelope
  and the parsed per-type structural contract.
- **vocabulary/** — the closed `status` and relationship enums as tables.
- **conformance/** — conformant/gated corpus and producer/consumer
  definitions, plus the OKF-consumer compatibility note.
- **examples/** — the Appendix A minimal corpus, one valid artifact per
  type, and one annotated invalid case per major rule, indexed by
  `manifest.json` as an executable acceptance suite.

One deliberate forward-looking addition beyond current validator behavior:
the corpus-level spec-version declaration (`rac_spec` in `.rac/config.yaml`,
SPEC.md §10.1), flagged in `DISCREPANCIES.md` pending reference-implementation
support.
