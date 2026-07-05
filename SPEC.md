# The RAC (Requirements as Code) Specification

**Version 0.1.0**

## 1. Abstract

RAC (Requirements as Code) is a format for typed, validated, versioned product
knowledge — requirements, decisions, roadmaps, prompts, and designs — stored as
Markdown files in a Git tree. This specification defines the artifact model,
the closed `status` lifecycle, the closed typed-relationship vocabulary, and
the validation and conformance rules a corpus and its tools must satisfy. It is
written for implementers of producers (tools that write RAC corpora) and
consumers (tools that read, validate, or serve them).

## 2. Status of this document

This is **v0.1.0** of the RAC specification, extracted from the reference
implementation, [`rac-core`](https://github.com/itsthelore/rac-core). Every
normative statement below is traceable to behavior the reference validator
enforces today (the machine-checked trace lives in `extraction-inventory.json`;
one deliberate exception is called out in §10). The specification is pre-1.0:
expect breaking changes before v1.0, governed by the compatibility policy in
§10. The specification versions independently of `rac-core`, which releases on
CalVer.

## 3. Motivation, goals, and non-goals

Coding agents re-introduce mistakes their teams already paid to rule out,
because the decision that ruled them out lives in prose an agent cannot be
trusted to interpret — or nowhere at all. RAC makes that knowledge a typed,
machine-validated corpus: every artifact has a type, an identity, a lifecycle
status, and explicitly typed links to the artifacts it relates to, and a CI
gate rejects corpus states that would mislead a reader — human or agent —
before they land.

**Goals.**

- A deterministic, offline-checkable definition of a valid corpus: the same
  corpus bytes always produce the same findings and the same exit code.
- Prescriptive-knowledge semantics: a closed lifecycle with supersession, and
  machine-readable relationship types with referential integrity.
- Implementation independence: everything here is checkable from the files
  alone, with no dependency on `rac-core`, Python, or any serving layer.

**Non-goals.**

- **Not a general knowledge format.** Portable descriptive knowledge — tables,
  metrics, runbooks — is OKF's job (§5); RAC composes with OKF rather than
  replacing it.
- **Not a retrieval or RAG system.** RAC defines what a corpus *is*, not how
  it is indexed, embedded, or served.
- **Not a project-management or ticketing replacement.** Status is knowledge
  currency (is this decision still current?), never work or delivery state;
  external tickets are referenced, not modeled.
- **Not a prescription of MCP or any particular serving layer.** The reference
  implementation ships an MCP server; nothing here requires one.

## 4. Terminology

> The key words MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD, SHOULD
> NOT, RECOMMENDED, MAY, and OPTIONAL in this document are to be interpreted
> as described in BCP 14 ([RFC 2119](https://www.rfc-editor.org/rfc/rfc2119),
> [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174)) when, and only when,
> they appear in all capitals.

- **Artifact** — one Markdown file carrying one unit of typed product
  knowledge: an optional YAML frontmatter envelope plus a structured body.
- **Corpus** — the set of artifacts under a corpus root directory, together
  with its root configuration file (`.rac/config.yaml`).
- **Type** — one of the closed set of artifact kinds (§6.1). A document
  matching no type is *untyped* ("unknown") — a valid outcome, not an error.
- **Status** — an artifact's lifecycle value, drawn from its type's closed
  enum (§7). Statuses partition into *live* and *retired*.
- **Typed link** — a reference from one artifact to another (or to an external
  target), declared in a relationship section whose name comes from the closed
  vocabulary in §8.
- **Supersession** — the lifecycle event in which an artifact is retired and,
  typically, replaced; the replacing decision points at the retired one via
  the `supersedes` edge (§7.2).
- **Finding** — one validation result, identified by a stable machine code,
  with an intrinsic *severity* and an *enforcement class* (§9).
- **Severity** — a finding's intrinsic annotation level: `error`, `warning`,
  or `info` (§9.1). Independent of whether the finding blocks.
- **Enforcement class** — whether a finding blocks the gate: `blocking` or
  `advisory` (§9.1). The exit signal fails iff a blocking finding is present.
- **Enforcement policy** — the corpus-committed mapping (in
  `.rac/config.yaml`) that MAY reclassify a finding's enforcement class or
  suppress it (§9.5). A *mechanism*, not part of spec conformance.
- **Default policy** — the enforcement classes with no policy overrides
  present (§9.3). Spec conformance is always evaluated under the default
  policy (§9.7); an unqualified conformance claim means the default policy.
- **Gate** — the composed enforcement run (structural validation, relationship
  integrity, and review) whose exit signal a CI check consumes (§9.4).
- **Tier** — a finding's review priority level, 1 (highest impact) through 6
  (§9.2).
- **Producer** — a tool that writes artifacts or corpora. **Consumer** — a
  tool that reads, validates, or serves them.

## 5. Relationship to OKF

RAC uses the same carrier as Google's Open Knowledge Format (OKF, v0.1 Draft):
a Git tree of Markdown files with YAML frontmatter. The relationship is
mechanical and one-directional:

1. A RAC corpus is expressible as a conformant OKF v0.1 bundle. A conformant
   producer MUST be able to emit the derived bundle: one file per typed
   artifact whose OKF `type` maps from the RAC type (`requirement`→`Requirement`,
   `decision`→`ADR`, `design`→`Design`, `roadmap`→`Roadmap`,
   `prompt`→`Prompt`), plus generated `index.md`/`log.md` entry points. A
   registered RAC type without an OKF mapping is a conformance error
   (`okf-unmapped-type`), never a silent omission. <!-- inv: okf_composition -->
2. RAC's other envelope keys (`schema_version`, `id`, `relationships`, `tags`)
   ride as OKF producer-defined keys, which OKF consumers are directed to
   preserve and not reject (OKF §9), so the export is lossless.
3. RAC identity is the explicit `id` field, not the file path (§6.4): a
   RAC-aware consumer MUST resolve links by identifier. OKF's path-as-identity
   does not survive a file rename; RAC's does.
4. The reverse direction is not defined — an arbitrary OKF bundle has untyped
   links, an open `type` set, and no lifecycle, so it cannot in general become
   a valid RAC corpus without human classification. That asymmetry is the
   point: RAC is the strict semantic layer over the permissive carrier.

A typed artifact MUST NOT be named `index.md` or `log.md`
(`okf-reserved-filename-collision`) — those collide with the generated bundle
entry points; an untyped document at those paths is a legitimate entry point.
<!-- inv: reserved_structure -->

## 6. Artifact model

### 6.1 Types

The artifact type set is closed: <!-- inv: artifact_types -->

`requirement` · `decision` · `roadmap` · `prompt` · `design`

A consumer MUST NOT invent types, and a producer MUST NOT emit a frontmatter
`type` outside this set (`invalid-metadata-field`). A document that declares
no type and matches no type structurally is *untyped*: consumers MUST treat it
as a valid, skipped document — never validate it against a type it does not
have, and never fail the corpus because of it. <!-- inv: classification -->

Classification is deterministic and structural: a document is scored against
each type by the `##` sections it contains (required sections count 1 point,
recommended ½; fit = points ÷ ceiling), and it classifies as the best-fit type
when fit ≥ 0.5 and at least one required section matched. Classification is
separate from validation: an invalid but recognizable artifact still
classifies as its type, then fails validation. <!-- inv: classification -->

### 6.2 File format

An artifact is one UTF-8 Markdown file: an OPTIONAL leading YAML frontmatter
block delimited by `---` (the closer MAY be `---` or `...`), then a CommonMark
body. The body MUST contain exactly one top-level `#` title
(`missing-title` / `multiple-titles`). Structure below the title is expressed
in `##` sections, matched case-insensitively and whitespace-trimmed.
<!-- inv: file_format -->

A corpus is discovered as the sorted set of `*.md` files under the corpus
root, recursively, skipping any path component that starts with `.`. No
directory layout is normative; `decisions/`, `requirements/` and similar are
conventions only. <!-- inv: reserved_structure -->

### 6.3 The frontmatter envelope

The frontmatter field set is closed: <!-- inv: frontmatter -->

| Field | Type | Presence | Rule |
| --- | --- | --- | --- |
| `schema_version` | integer | REQUIRED when frontmatter is present | MUST be a supported version (this spec: `1`); anything else is `unsupported-schema-version` |
| `id` | string | RECOMMENDED | MUST match the ID grammar (§6.4); normalized uppercase |
| `type` | string | RECOMMENDED | MUST be a registered type (§6.1) |
| `relationships` | map | OPTIONAL (reserved) | kind → list of ID strings; shape-validated; not yet consumed (§8.6) |
| `tags` | list | OPTIONAL | non-empty strings; descriptive labels only |

An unknown frontmatter field is an error (`invalid-metadata-field`), never
ignored. Malformed YAML, duplicate keys, YAML aliases, nesting beyond 32
levels, and blocks over 64 KiB are all errors (`malformed-frontmatter`,
`duplicate-frontmatter-key`). A document with *no* frontmatter block at all is
legal (the legacy form). Timestamps are deliberately absent: recency is
derived from version control, never stored in frontmatter.
<!-- inv: frontmatter --> <!-- inv: limits -->

```markdown
---
schema_version: 1
id: RAC-01JY4M8X2QZ7
type: decision
---
# ADR-004: Parser Strategy
```

Product reasoning — status, context, decisions, requirements — lives in body
sections, and MUST NOT migrate into the envelope. The envelope is
machine-operational metadata only.

### 6.4 Identity

An artifact ID is `<KEY>-<SUFFIX>`: a repository key matching
`^[A-Z][A-Z0-9]{1,9}$` (declared once in the corpus config, §6.5), a hyphen,
and a 12-character Crockford base32 suffix (uppercase, no I/L/O/U):
<!-- inv: id_rules -->

```text
^[A-Z][A-Z0-9]{1,9}-[0-9A-HJKMNP-TV-Z]{12}$     e.g. RAC-01JY4M8X2QZ7
```

IDs are matched case-insensitively and normalized to uppercase. Generation is
offline and branch-safe (an 8-character millisecond-timestamp segment plus a
4-character random segment); no allocation server exists or is permitted.

**Identity resolution precedence** (first match wins): the frontmatter `id`;
an explicit `## ID` section value; a `<letters>-<digits>` prefix of the
filename stem (e.g. `adr-004` from `adr-004-parser-strategy.md`); the whole
filename stem. An artifact answers to *all* of these as resolution aliases, so
human-readable references (`ADR-004`) keep resolving after an artifact adopts
a canonical ID. The document title is never an identifier, and inline
`[REQ-NNN]` requirement lines are not identifiers — link targets are whole
artifacts. <!-- inv: id_rules -->

Three normative consequences:

- **Identity survives file moves.** The canonical identity is the declared
  `id`, never the path. Renaming or moving a file MUST NOT change the
  artifact's identity, and inbound references MUST keep resolving.
- **Uniqueness is corpus-wide on the canonical identifier.**
  `duplicate-artifact-identifier` is a blocking finding. Aliases never create
  duplicates on their own.
- **Conflicts are never silently resolved.** A frontmatter `id` that differs
  from a declared legacy identity is `conflicting-identity` (error); a
  consumer MUST NOT pick one.

### 6.5 The corpus configuration

A corpus root holds `.rac/config.yaml`, discovered by walking upward from the
working directory. It carries machine policy, not knowledge:
`repository_key` (REQUIRED for ID generation), `rac_spec` (the spec-version
declaration, §10.1), and OPTIONAL `ticketing.provider`, `validation` severity
overrides, and `enforcement` policy (§9.5). <!-- inv: reserved_structure -->

### 6.6 Sections per type

For each type: required sections MUST be present (a missing one is the
blocking finding `missing-<section>`); recommended sections never fail
(their absence is a tier-4 advisory, §9.2); optional sections are recognized
and extracted but never scored and never reported missing.
<!-- inv: sections_per_type -->

| Type | Required sections | Recommended (in priority order) |
| --- | --- | --- |
| `requirement` | Problem, Requirements | Success Metrics, Risks, Assumptions |
| `decision` | Context, Decision, Consequences | Status, Category, Alternatives Considered |
| `roadmap` | Outcomes, Initiatives | Success Measures, Assumptions, Risks |
| `prompt` | Objective, Input, Instructions, Output | Constraints, Examples, Evaluation |
| `design` | Context, User Need, Design, Constraints | Rationale, Alternatives, Accessibility, Style Guidance, Open Questions |

Each type's optional sections are exactly its relationship sections (§8.2)
plus, for `decision`, `Supersedes`. Recognized heading synonyms (e.g.
`Success Criteria` → `Success Metrics`) aid classification only; validation
expects the canonical headings. <!-- inv: sections_per_type -->

### 6.7 Requirement lines

Inside a `requirement` artifact's `## Requirements` section, each non-empty
line MUST be a well-formed requirement statement: <!-- inv: requirement_line_grammar -->

```text
[REQ-<digits>] <non-empty description>
```

A line with no bracket prefix is `req-missing-id`; a malformed bracket ID is
`malformed-req-id`; an empty description is `empty-req-text`; a duplicate
`REQ` ID within the file is `duplicate-req-id` — all blocking. Requirement-ID
uniqueness is per-file.

Normative language inside requirement lines is disciplined per BCP 14: a
lowercase or mixed-case `shall`/`must`/`should` is ambiguous and blocking
(`requirement-normative-keyword`), because only ALL-CAPS keywords carry
normative weight. <!-- inv: requirement_line_grammar -->

> **Not specified.** The reference implementation also emits *advisory*
> ISO/IEC/IEEE 29148 "singular requirement" and EARS response-clause findings
> (`requirement-not-singular`, `requirement-non-ears`, `requirement-ears-clause`;
> ADR-056). These are authoring heuristics, not conformance rules, and are
> deliberately excluded — only the BCP-14 casing rule above is normative.

```markdown
## Requirements

- [REQ-001] The exporter MUST emit one bundle entry per typed artifact.
- [REQ-002] If the corpus is empty, then the exporter SHALL emit an empty index.
```

### 6.8 Machine-readable schemas

The JSON Schemas in `schema/` are the normative machine-readable form of
§6.3–§6.7: `schema/frontmatter.schema.json` for the envelope,
`schema/artifact.schema.json` for the parsed per-type structural contract.
They are documentation for implementers, derived from validator behavior — the
reference implementation itself ships no JSON Schema and takes no
schema-validation dependency, so these are not a `rac-core` artifact. Prose
and schema are cross-checked against one extraction inventory; a disagreement
is a defect in this specification (file an issue), not a license to pick one.
Prose-only specs drift: OKF shipped prose-only and its own reference parser
diverged from the text within days (it requires four frontmatter fields; the
text requires one — see `conformance/conformance.md`).

## 7. The `status` lifecycle

Status is declared in a `## Status` body section — the first non-empty line is
the value, matched case-insensitively. Status is OPTIONAL: a missing section
is always legal. A *present* value MUST belong to the type's closed enum; an
unknown value is the blocking finding `invalid-<type>-status`.
<!-- inv: status_enums -->

| Type | Live values | Retired values |
| --- | --- | --- |
| `requirement` | Proposed, Accepted | Superseded, Deprecated |
| `decision` | Proposed, Accepted | Superseded, Deprecated |
| `design` | Proposed, Accepted | Superseded, Deprecated |
| `roadmap` | Planned, Achieved | Superseded, Abandoned |
| `prompt` | Active | Deprecated |

`Achieved` (roadmap) is a live *terminal* state: the intent was delivered, the
record remains valid to reference. Retirement marks replaced or dropped
knowledge, not completed work — status is knowledge currency, never delivery
tracking. <!-- inv: status_enums -->

No state-machine transitions are enforced: any enum value may be written at
any time, and history lives in version control. The lifecycle's force comes
entirely from the live/retired partition:

### 7.1 Two other constrained enums

`decision` MAY declare `## Category` (Architecture, Product, Process,
Technical, Other; unknown value → `invalid-decision-category`). `roadmap` MAY
declare `## Horizon` (`now`, `next`, `later`, or a quarter matching
`Q[1-4] YYYY`; unknown value → `invalid-roadmap-horizon`). Both blocking.
<!-- inv: status_enums -->

### 7.2 Supersession semantics

When an artifact carries a retired status:

- A **live** artifact MUST NOT reference it through any relationship except
  `supersedes` — such a reference is the finding
  `relationship-target-superseded`, blocking by default (§9.3).
- The `supersedes` edge is exempt by design: the replacing decision
  legitimately points at the decision it retires.
- References *from* retired artifacts are exempt: historical chains stay
  intact.
- A consumer MUST NOT present a retired artifact as current knowledge
  (conformant-consumer rule, §9.6).

The retired decision (`RAC-01JYAAAAAAAA`) sets `## Status` to `Superseded`;
the replacing decision names it in its own body, the one edge allowed to point
at a retired target:

```markdown
## Supersedes

- RAC-01JYAAAAAAAA
```

## 8. Typed relationships

### 8.1 Design position

OKF §5.3 states that the specific kind of relationship a link asserts
"is conveyed by the surrounding prose, not by the link itself," and that
consumers must tolerate broken links. RAC deliberately inverts both choices:
relationship semantics are machine-readable, and broken links are blocking.
Agents cannot be trusted to infer edge types from prose — that inference gap
is precisely the failure RAC exists to close. A link whose meaning lives in
prose is a link an agent will misread; a link whose target silently dangles is
a decision an agent will never find.

### 8.2 The vocabulary

Relationships are declared as body sections whose names come from a closed
vocabulary. One reference per non-empty line; a leading list marker is
stripped; otherwise the line text *is* the reference, preserved verbatim.
<!-- inv: relationship_types -->

| Edge | Declared by | Target (range) | Direction | Validation |
| --- | --- | --- | --- | --- |
| `related_requirements` | all five types | `requirement` | undirected | resolve + range + status |
| `related_decisions` | all five types | `decision` | undirected | resolve + range + status |
| `related_roadmaps` | all five types | `roadmap` | undirected | resolve + range + status |
| `related_prompts` | requirement, roadmap, design | `prompt` | undirected | resolve + range + status |
| `related_designs` | requirement, decision, roadmap, prompt | `design` | undirected | resolve + range + status |
| `supersedes` | decision | `decision` | directed (inverse `superseded-by`), acyclic | resolve + range; exempt from the retired-target rule |
| `related_tickets` | all five types | external ticket key/URL | undirected | format-linted against the configured provider; never resolved |
| `verified_by` | requirement | external test/trace path | directed (inverse `verifies`) | recorded; not resolved, not existence-checked |
| `applies_to` | decision | repo path, glob, or component | directed (inverse `governed_by`) | literal path entries existence-checked against the repo root |

Section headings map to edge names by lowercasing and replacing spaces with
underscores (`## Related Decisions` → `related_decisions`). Cardinality is
many-valued for every edge. Edge legality by source type ("declared by") is
enforced: a vocabulary section populated on a type that does not declare it is
`relationship-edge-unsupported` and produces no edge. <!-- inv: relationship_types -->

```markdown
## Related Decisions

- RAC-01JYBBBBBBBB
- ADR-016
```

### 8.3 Referential integrity

Every non-external reference MUST resolve, uniquely, to another artifact in
the corpus, via the identity aliases of §6.4:

- no match → `relationship-target-not-found` (blocking);
- more than one match → `relationship-target-ambiguous` (blocking);
- resolves to the declaring artifact → `relationship-self-reference`
  (blocking by default).

A resolved target MUST be of the edge's range type
(`relationship-target-type-mismatch`, blocking); an untyped target is not a
range violation. A resolved live-to-retired reference outside `supersedes` is
`relationship-target-superseded` (§7.2). <!-- inv: relationship_types -->

### 8.4 Graph shape

The `supersedes` graph MUST be acyclic. A cycle — reported once per strongly
connected component — is `relationship-cycle` (blocking). Undirected
`related_*` edges carry no shape constraint. <!-- inv: relationship_types -->

### 8.5 External edges

`related_tickets`, `verified_by`, and `applies_to` target things outside the
corpus, so they are exempt from resolution, range, and status checks. Ticket
references are format-linted offline against the corpus-configured provider
(`jira`, `github`, `linear`, `azure-devops`, `servicenow`, or `none`;
malformed entry → `malformed-ticket-reference`, blocking). `applies_to`
literal path entries (containing `/`, no glob characters) MUST exist relative
to the repository root (`applies-to-target-not-found`, blocking); glob and
component-label entries are recorded without checking. The engine never
contacts an external system: all checks are pure functions of the corpus and
the working tree. <!-- inv: ticketing_providers --> <!-- inv: relationship_types -->

### 8.6 Reserved: frontmatter relationships

The frontmatter `relationships` map (§6.3) is reserved for a future migration
of relationship declarations into the envelope. Producers MAY write it (shape:
edge name → list of canonical IDs); consumers MUST parse and preserve it but
MUST NOT yet treat it as the source of relationship truth — the body sections
are authoritative. <!-- inv: frontmatter -->

## 9. Validation, severity, and conformance levels

### 9.1 The finding model

Every check has a stable machine code. A finding carries two independent
classifications: <!-- inv: severity_model -->

- **Intrinsic severity** — `error` | `warning` | `info`. This drives the
  annotation level in reports (SARIF: `error` / `warning` / `note`).
- **Enforcement class** — `blocking` | `advisory`. This drives the exit
  signal: a gate run fails if and only if at least one blocking finding is
  present. The two are independent: a warning-severity finding can be blocking
  (`relationship-target-superseded` is).

All findings MUST be reported; advisory findings MUST NOT affect conformance
or the exit signal.

### 9.2 Review tiers

Aggregated review classifies findings into six priority tiers. **Tiers 1 and 2
render a corpus non-conformant; a conformant CI gate MUST fail on any tier 1
or tier 2 finding. Tiers 3–6 MUST be reported but MUST NOT affect
conformance.** <!-- inv: severity_model -->

| Tier | Name (as in the reference implementation) | Code | Fails? |
| --- | --- | --- | --- |
| 1 | Invalid artifact (validation errors) | `invalid-artifact` | **yes** |
| 2 | Broken relationship | `broken-relationship` | **yes** |
| 3 | Missing required information (unrecognized artifact) | `unknown-artifact` | no |
| 4 | Missing recommended information | `missing-recommended-sections` | no |
| 5 | Stale corpus (write-cadence nudge; opt-in) | `stale-corpus` | no |
| 6 | Suspect drift (referenced target changed later) | `suspect-artifact` | no |

Tiers 5 and 6 are advisory-only extensions that give version-control-derived
hygiene signals a reporting slot without ever gating.

### 9.3 The normative check table

Every specified check, with its intrinsic severity and default enforcement
class. This table is generated from `extraction-inventory.json` (`checks`) and
MUST stay in lockstep with it. <!-- inv: checks -->

**Structural validation (source: `validate`) — errors are blocking, warnings
and info advisory:**

| Code | Severity | Enforcement |
| --- | --- | --- |
| `artifact-oversize`, `unreadable-artifact` | error | blocking |
| `malformed-frontmatter`, `duplicate-frontmatter-key`, `invalid-metadata-field`, `unsupported-schema-version`, `invalid-id-syntax`, `conflicting-identity` | error | blocking |
| `missing-title`, `multiple-titles` | error | blocking |
| `missing-<required-section>` (per type, §6.6) | error | blocking |
| `invalid-requirement-status`, `invalid-decision-status`, `invalid-decision-category`, `invalid-roadmap-status`, `invalid-roadmap-horizon`, `invalid-prompt-status`, `invalid-design-status` | error | blocking |
| `req-missing-id`, `empty-req-text`, `malformed-req-id`, `duplicate-req-id` | error | blocking |
| `requirement-normative-keyword` | error | blocking |
| `malformed-ticket-reference` | error | blocking |
| `okf-unmapped-type`, `okf-reserved-filename-collision` | error | blocking |
| `non-utf8-content`, `field-truncated`, `body-truncated` | warning | advisory |
| `roadmap-no-advancement-link` | warning | advisory |
| `missing-success-metrics`, `missing-risks`, `empty-problem`, `too-many-requirements`, `duplicate-req-text`, `ambiguous-verb` | warning | advisory |

**Relationship integrity (source: `relationships`) — every finding is blocking
by default, regardless of intrinsic severity:**

| Code | Severity | Enforcement |
| --- | --- | --- |
| `duplicate-artifact-identifier` | error | blocking |
| `relationship-target-not-found` | error | blocking |
| `relationship-target-ambiguous` | error | blocking |
| `relationship-target-type-mismatch` | error | blocking |
| `relationship-cycle` | error | blocking |
| `applies-to-target-not-found` | error | blocking |
| `relationship-target-superseded` | warning | blocking |
| `relationship-self-reference` | warning | blocking |
| `relationship-edge-unsupported` | warning | blocking |

**Review (source: `review`) — tiers 1–2 blocking, 3–6 advisory:**

| Code | Tier | Severity | Enforcement |
| --- | --- | --- | --- |
| `invalid-artifact` | 1 | error | blocking |
| `broken-relationship` | 2 | warning | blocking |
| `unknown-artifact` | 3 | info | advisory |
| `missing-recommended-sections` | 4 | warning | advisory |
| `stale-corpus` | 5 | info | advisory |
| `suspect-artifact` | 6 | warning | advisory |

### 9.4 The gate and its exit contract

The gate composes structural validation (including OKF conformance),
relationship integrity, and review over one corpus snapshot, applies the
corpus enforcement policy (§9.5), and reports one deterministic, sorted
finding list. Exit contract: <!-- inv: gate_semantics --> <!-- inv: exit_codes -->

| Exit | Meaning |
| --- | --- |
| 0 | no blocking finding |
| 1 | at least one blocking finding |
| 2 | usage or I/O error |

A conformant CI integration MUST fail its check exactly when the gate exits
non-zero, and MUST NOT reinterpret findings or decide independently what
blocks: enforcement classification belongs to the engine under the committed
corpus policy. Reports MUST be deterministic — the same corpus bytes **and
the same committed configuration** yield a byte-identical report (stable sort,
no timestamps). In SARIF output the level encodes the *intrinsic severity*
(`error`/`warning`/`note`); the enforcement class is carried solely by the
exit code.

### 9.5 Policy: overrides are a governed, committed mechanism

The classifications in §9.3 are the **default policy** — the enforcement
classes with no overrides present. A corpus MAY tune them in its committed
configuration: per-rule/per-type severity overrides (`validation:` — downgrade
`error` → `warning`, or `off` to suppress) and per-code enforcement
reclassification (`enforcement:` — `blocking` / `advisory` / `off`; precedence
`off` > `blocking` > `advisory` > default). This is a *mechanism*, not part of
spec conformance: an override changes the local gate verdict, never what §9.3
specifies. Because the policy is a committed file, the same repository state
always yields the same findings and exit code, and a tool MUST NOT apply
enforcement policy from any source outside the corpus.
<!-- inv: severity_model -->

### 9.6 What a conformant consumer MUST reject

OKF §9 enumerates what consumers must *not* reject over — unknown types,
missing optional fields, broken cross-links, unknown keys, missing indexes.
RAC's strict mirror, same device, inverted content — a conformant consumer
MUST reject (refuse to treat as a valid corpus):

1. any tier 1 or tier 2 finding — equivalently, any blocking finding under the
   corpus's committed policy;
2. any unknown `status` value, and any populated relationship section whose
   edge is outside the declaring type's vocabulary — fail, do not guess;
3. any reference from a live artifact to a retired artifact presented as
   current (outside `supersedes`);
4. any corpus declaring a spec version newer than the consumer supports
   (§10.3) — and, per artifact, any `schema_version` outside the supported
   set.

The inversion of OKF's permissiveness is deliberate. Descriptive knowledge
degrades gracefully when stale: an agent reading a slightly outdated table
description writes slightly worse queries. Prescriptive knowledge does not:
an agent citing a superseded architecture decision re-introduces the exact
mistake the team already paid to rule out. A consumer that guesses on an
unknown enum value is worse than one that fails: its output is
indistinguishable from grounded knowledge.

### 9.7 Conformance levels

Conformance is always evaluated under the **default policy** (§9.5): local
severity overrides and enforcement reclassifications change a repository's own
gate verdict, never its spec conformance — which is what makes a conformance
claim portable. A claim MUST name the policy it was evaluated under; an
unqualified claim means the default policy.

- **Conformant corpus** — under the default policy, structural validation
  (§9.3, `validate` source) yields zero blocking findings, and every
  artifact-level rule in §6–§7 holds.
- **Gated corpus** — a conformant corpus whose full gate run (§9.4) yields
  zero blocking findings under the default policy: link integrity holds, no
  live artifact references a retired one outside `supersedes`, and review
  tiers 1–2 are empty.
- **Conformant producer** — emits only artifacts that are (individually and
  jointly) a conformant corpus, and never emits an enum value, frontmatter
  field, or edge name outside this specification (extend via §11, never via
  enum values).
- **Conformant consumer** — resolves identity per §6.4, preserves the
  extension content it round-trips (§11), refuses everything in §9.6, and
  MUST NOT present a retired artifact as current.

## 10. Versioning and compatibility policy

This specification uses semantic versioning, independent of any
implementation's release scheme.

### 10.1 Corpus-level spec-version declaration

Every corpus MUST declare the specification version it targets, in one
machine-readable location: the `rac_spec` key of the corpus root
configuration, `.rac/config.yaml`: <!-- inv: spec_version_declaration -->

```yaml
repository_key: RAC
rac_spec: "0.1"
```

**`rac_spec` versus `schema_version`.** These are two different version axes
and MUST NOT be conflated. `rac_spec` (corpus-level, this section) declares
which version of *this specification* — the whole contract of §6–§11 — a
corpus targets. `schema_version` (per-artifact frontmatter, §6.3) declares
which version of the *frontmatter envelope* an individual artifact uses. Each
spec version states which envelope versions it accepts: **spec v0.1 accepts
`schema_version: 1`, and no other.** A future spec version MAY accept
additional envelope versions; that mapping is part of the spec contract, so a
consumer resolves envelope compatibility only after it has accepted the
declared `rac_spec` (§10.3).

The declaration lives in the corpus config, uniformly, no exceptions. OKF §11
puts its `okf_version` declaration in the frontmatter of `index.md` — a file
OKF otherwise defines as a generated, frontmatter-free entry point; that
special case is exactly the kind of parser wart this specification refuses to
copy.

> **Implementation status.** This subsection is the one deliberate
> forward-looking addition in this specification. At ratification the
> maintainer committed `rac-core` to implementing it — reading `rac_spec`,
> refusing newer-versioned corpora per §10.3, and declaring the key in its
> own corpus — before v0.1.0 is announced; the MUST stands on that basis.
> Until that lands, the reference implementation ignores the key (which is
> harmless: unknown corpus-config keys are not errors), and this note is the
> record of the gap.

### 10.2 What may change when

- New enum values (a `status` value, a relationship type, an artifact type)
  MAY be added in a **minor** version.
- Removing or changing the meaning of an existing enum value, changing an
  existing check's tier or default enforcement class, or tightening a MUST,
  REQUIRES a **major** version.
- New advisory checks MAY be added in a minor version; new *blocking* checks
  REQUIRE a major version.
- Editorial changes and new examples are **patch** versions.

### 10.3 Version-mismatch behavior

A consumer encountering a declared spec version newer than it supports MUST
fail with a diagnostic naming both versions ("corpus targets 0.3, consumer
supports 0.1"); it MUST NOT attempt best-effort interpretation. Contrast OKF
§11, which pairs its version declaration with a SHOULD for best-effort
consumption by consumers that do not understand the version — rendering the
declaration nearly informational. RAC pairs the declaration with mandatory
refusal, which makes it load-bearing. The field is not the differentiator;
the failure semantics are.

The rationale is diagnosability: strict unknown-value handling (§9.6) is only
actionable if a consumer can distinguish a corrupt corpus from one targeting a
newer spec — which is why strict ecosystems pair failure with a version field
(JSON Schema's `$schema`, OpenAPI's version). Accordingly, an unknown-enum
error MUST include the declared spec version, so it reads "corpus targets 0.3,
consumer supports 0.1," never an opaque rejection.

## 11. Extension points

Producers extend RAC via **new keys and sections, never new values in the
closed enums**:

- **`tags`** carries arbitrary descriptive labels — this is the extension
  surface inside the frontmatter envelope. The envelope's field set is
  otherwise closed (§6.3): a producer MUST NOT add its own frontmatter keys
  (`invalid-metadata-field`). A namespaced extension-key scheme MAY be
  specified in a future minor version; until then, structured extension data
  belongs in body sections.
- **Non-vocabulary body sections** are free: any `##` section outside a
  type's required/recommended/optional set is carried, searchable content
  with no validation semantics.
- Consumers MUST preserve extension content they round-trip (`tags`, the
  `relationships` map, free body sections) so the OKF export stays lossless.
- What extension MUST NOT do: add a `status` or `category` value, a
  relationship section name, or an artifact type. Those are enum changes and
  follow §10.2.

## Appendix A — Minimal complete corpus

The smallest corpus that validates with zero blocking findings — config with
the spec-version declaration, one decision, one requirement, one typed link —
also shipped (recommended sections filled in) as `examples/minimal-corpus/`:

```text
minimal/
├── .rac/
│   └── config.yaml
├── record-decisions-as-adrs.md
└── decision-capture.md
```

`.rac/config.yaml`:

```yaml
repository_key: MIN
rac_spec: "0.1"
```

`record-decisions-as-adrs.md`:

```markdown
---
schema_version: 1
id: MIN-01JYX0000001
type: decision
---
# ADR-001: Record decisions as ADRs

## Status

Accepted

## Context

Decisions were scattered across chat and memory, so the team repeated
debates it had already settled.

## Decision

Every significant decision is recorded as an ADR in this corpus.

## Consequences

Decisions become citable and supersedable; writing them takes effort.

## Related Requirements

- MIN-01JYX0000002
```

`decision-capture.md`:

```markdown
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

## Related Decisions

- MIN-01JYX0000001
```

(Recommended sections are omitted here for brevity — their absence is
advisory, never blocking.)

---

*This specification is maintained at
[`itsthelore/rac-spec`](https://github.com/itsthelore/rac-spec) under the
process in `CONTRIBUTING.md` and `GOVERNANCE.md`. The reference implementation
is [`itsthelore/rac-core`](https://github.com/itsthelore/rac-core).*
