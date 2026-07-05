# RAC (Requirements as Code) — the relationship vocabulary

Normative reference: [SPEC.md §8](../SPEC.md). The vocabulary is closed.
Relationships are declared as `##` body sections; the section heading maps to
the edge name by lowercasing and replacing spaces with underscores
(`## Related Decisions` → `related_decisions`). Each non-empty line of the
section body is one reference (a leading Markdown list marker is stripped;
the remaining line text *is* the reference, verbatim).

## The edges

| Edge | Declared by (domain) | Target (range) | Direction | Inverse | Cardinality | Validation level |
| --- | --- | --- | --- | --- | --- | --- |
| `related_requirements` | all five types | `requirement` | undirected | — | many | resolve + range + status: error/blocking on failure |
| `related_decisions` | all five types | `decision` | undirected | — | many | resolve + range + status |
| `related_roadmaps` | all five types | `roadmap` | undirected | — | many | resolve + range + status |
| `related_prompts` | requirement, roadmap, design | `prompt` | undirected | — | many | resolve + range + status |
| `related_designs` | requirement, decision, roadmap, prompt | `design` | undirected | — | many | resolve + range + status |
| `supersedes` | decision | `decision` | **directed, acyclic** | `superseded-by` | many | resolve + range + acyclicity; exempt from the retired-target rule |
| `related_tickets` | all five types | external ticket key or URL | undirected | — | many | format lint against the configured provider (`malformed-ticket-reference`, blocking); never resolved in-corpus |
| `verified_by` | requirement | external test/trace file path | directed | `verifies` | many | recorded only; not resolved, not existence-checked |
| `applies_to` | decision | repository path, glob, or component label | directed | `governed_by` | many | literal path entries existence-checked (`applies-to-target-not-found`, blocking); globs and labels recorded only |

Cardinality is declared (many) and not otherwise enforced. Inverse labels are
display semantics for consumers rendering the graph; only the forward
direction is authored.

## Findings

| Code | Trigger | Intrinsic severity | Default enforcement |
| --- | --- | --- | --- |
| `relationship-target-not-found` | reference resolves to nothing | error | blocking |
| `relationship-target-ambiguous` | reference resolves to more than one artifact | error | blocking |
| `relationship-target-type-mismatch` | resolved target's type outside the edge's range (untyped targets exempt) | error | blocking |
| `relationship-cycle` | cycle in an acyclic edge kind (`supersedes`); one finding per strongly connected component | error | blocking |
| `duplicate-artifact-identifier` | two artifacts share a canonical identifier | error | blocking |
| `applies-to-target-not-found` | literal `applies_to` path missing from the working tree | error | blocking |
| `relationship-target-superseded` | live artifact references a retired one outside `supersedes` | warning | **blocking** |
| `relationship-self-reference` | reference resolves to the declaring artifact | warning | **blocking** |
| `relationship-edge-unsupported` | vocabulary section populated on a type that does not declare it | warning | **blocking** |

Note the last three: warning-*severity* but blocking by default — the
annotation level and the enforcement class are independent (SPEC.md §9.1).
Every relationship finding fails `rac relationships --validate` (exit 1)
regardless of severity; a corpus policy may reclassify a specific code
(SPEC.md §9.5).

## Out-of-vocabulary sections

A populated **vocabulary** section on a type that does not declare it is
`relationship-edge-unsupported`. A `##` section whose name is **outside** the
vocabulary entirely (e.g. `## Related Widgets`) is ordinary body content: no
edge and no finding — it is indistinguishable from prose. Extension is via
`tags` or free body sections, never via new relationship section names
(SPEC.md §11); adding an
edge to this vocabulary is a **minor** spec version, removing or re-defining
one a **major** version.
