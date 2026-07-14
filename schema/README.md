# Schemas

## The shared registry engines read

**`artifact-specs.json`** is the canonical machine-readable artifact-spec
registry: the ordered artifact specs (requirement, decision, roadmap, prompt,
design) with their section tiers, metadata enums, descriptions, guidance,
synonyms, and starter bodies, plus the relationship-section descriptions.
Unlike the JSON Schemas below, it is not derived documentation — it is a
source of truth an engine reads directly. The reference implementation
([`rac-core`](https://github.com/itsthelore/rac-core)) vendors it and both of
its engines (Python and Rust) load their artifact specs from the vendored
copy; a sync gate keeps that copy identical to this file (rac-core ADR-115,
closing ADR-063 Guard 1). Changes to it are normative and follow the process
in [CONTRIBUTING.md](../CONTRIBUTING.md).

## The JSON Schemas

These JSON Schemas pin the **single-document structural layer** of RAC
(Requirements as Code) — and only that layer. They are the machine-readable
form of [SPEC.md](../SPEC.md) §6.3–§6.7:

- **`frontmatter.schema.json`** — the frontmatter envelope: the closed field
  set, `schema_version`, the ID grammar, the `type` enum, and the shape of
  `relationships`/`tags`.
- **`artifact.schema.json`** — the parsed per-type structural contract: a
  discriminated union over `type` requiring each type's sections and
  constraining its `## Status`/`## Category`/`## Horizon` enum values.

## What these schemas cannot express

The rules that make RAC *RAC* are properties of a **corpus**, not of one file,
so they are normative in SPEC.md and are not, and cannot be, encoded here:

- **Referential integrity** (§8.3) — a reference resolving uniquely to another
  artifact of the edge's range type.
- **The `status` lifecycle and supersession** (§7) — a live artifact not
  referencing a retired one except via `supersedes`.
- **Graph shape** (§8.4) — the `supersedes` graph being acyclic.
- **Identity** (§6.4) — corpus-wide id uniqueness, alias resolution, and
  path-independence.
- **The finding model** (§9) — intrinsic severity vs. enforcement class, the
  default-policy classification table, conformance levels, and consumer
  behavior (refusing to present a superseded artifact as current).
- **Versioning and compatibility** (§10) — the `rac_spec` declaration,
  mandatory version refusal, and what may change in a minor vs. major release.

A validator that ran only these schemas would accept corpora RAC rejects.
SPEC.md is the contract; these files restate one layer of it. The reference
implementation (`rac-core`) ships no JSON Schema at all (its validator is
imperative code, ADR-052), so these files are derived documentation for
implementers, not a dependency of any tool.
