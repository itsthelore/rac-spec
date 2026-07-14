# RAC (Requirements as Code) — conformance

Normative reference: [SPEC.md §9](../SPEC.md). This page restates the
conformance targets an implementation claims against, and how to test a
claim. RFC 2119/8174 keywords carry their SPEC.md meaning here.

## Conformance targets

### Conformant corpus

A corpus is **conformant** when structural validation produces zero blocking
findings: every artifact parses, the frontmatter envelope is well-formed
(closed field set, supported `schema_version`, valid `id` syntax, no identity
conflicts), every typed artifact has its type's required sections, every
present constrained enum value (`status`, `category`, `horizon`) is in its
closed set, requirement lines are well-formed `[REQ-N]` statements, and the
OKF carrier rules hold (no typed artifact named `index.md`/`log.md`, every
type mapped). Untyped documents are skipped, not failed.

### Gated corpus

A **gated** corpus is a conformant corpus that also passes full gate
semantics with zero blocking findings:

- referential integrity — every non-external reference resolves uniquely to
  an in-corpus artifact of the edge's range type; no duplicate canonical
  identifiers; no `supersedes` cycles; no self-references;
- status consistency — no live artifact references a retired artifact except
  via `supersedes`;
- review tiers 1–2 are empty (no invalid artifacts, no broken relationships);
- declared `applies_to` literal paths exist in the working tree; ticket
  references are well-formed for the configured provider.

"Gated" is the level a CI merge gate certifies: the reference gate exits 0
exactly when a corpus is gated under its committed enforcement policy.

### Conformant producer

A tool that writes artifacts or corpora. It MUST:

- emit only artifacts that leave the corpus conformant (and SHOULD leave it
  gated);
- never emit an enum value, frontmatter field, or relationship section name
  outside the specification — extension is via SPEC.md §11 only;
- generate identifiers per the ID grammar, offline, with no shared allocation
  state;
- never rewrite an established identity or resolve an identity conflict
  silently;
- be able to emit the derived OKF bundle (SPEC.md §5) for interchange.

### Conformant consumer

A tool that reads, validates, or serves a corpus. It MUST:

- resolve references by identifier with the SPEC.md §6.4 precedence and
  aliases — never by file path;
- report all findings and fail on any blocking finding (tier 1–2);
- refuse to present a retired artifact as current knowledge;
- fail, rather than guess, on any enum value outside its known vocabulary, and on
  any corpus declaring a spec version newer than it supports, naming both
  versions in the diagnostic;
- preserve producer-defined content it round-trips (frontmatter
  `relationships`, `tags`, free body sections);
- apply enforcement policy only from the corpus's committed configuration.

A consumer MAY be read-only (no gate); it then claims conformance for
resolution, retirement handling, and refusal semantics only.

## Testing a claim

The `examples/` directory is the executable conformance surface:

- every corpus under `examples/valid/` and `examples/minimal-corpus/` MUST
  produce zero blocking findings;
- every case under `examples/invalid/` MUST produce the finding code named in
  `examples/manifest.json`, at the documented intrinsic severity and default
  enforcement class, and nothing weaker.

An implementation SHOULD run the manifest as its acceptance suite. The
reference implementation (`rac-core`) is the arbiter where this repository
and an implementation disagree — and where this repository and `rac-core`
disagree, that is a spec defect: file an issue rather than silently following
either.

### Output-parity tier

[`output-parity.json`](output-parity.json) defines a stricter, optional tier
for implementations that claim byte-for-byte output parity with the reference
implementation (the bar rac-core ADR-063 Guard 2 sets for a native engine
port). Each case pins an argv, an expected exit code, and a golden stdout
file in [`vectors/`](vectors/) with its sha256 — deterministic, recency-free
commands (`validate`, `inspect`, `relationships`, `stats`, all `--json`) run
from this repository's root over the example corpora, plus `validate --json`
on three invalid cases. The goldens are the arbiter's exact output. An
implementation conforms to this tier when, under the manifest's pinned
environment, it reproduces every case's stdout bytes and exit code exactly.
This tier is not required for producer/consumer conformance above; it exists
so a reimplementation can prove it is indistinguishable from the reference,
not merely conformant.

## Compatibility note: OKF consumers

SPEC.md §5 claims a RAC corpus exports to a conformant OKF v0.1 bundle. When
testing that claim against OKF tooling, note a known discrepancy in the OKF
ecosystem itself: Google's reference parser has been observed to require
`type`, `title`, `description`, and `timestamp`, although the OKF spec text
requires only `type`. RAC targets the **spec text**. Bundles exported per
SPEC.md §5 carry `type` (mapped), `id`, git-derived `created`/`updated`, and
`tags` when present; they do not carry a frontmatter `title` (OKF-optional; a
consumer without it MAY derive a title from the filename, OKF §4.1) or
`description`. A consumer pinned to the behavior of Google's
reference parser rather than the OKF spec text may therefore reject a valid
bundle; that is a defect in that parser, not in the bundle.
