# rac-spec — the specification for RAC (Requirements as Code)

*Typed, validated, versioned product knowledge for coding agents.*

RAC (Requirements as Code) is a format for storing a team's requirements,
decisions, roadmaps, prompts, and designs as Markdown artifacts in a Git
tree — with a closed lifecycle, machine-readable relationship types, and a CI
gate that rejects broken corpus states before they land.
[`rac-core`](https://github.com/itsthelore/rac-core) is the reference
implementation; this repository is the contract a second implementation would
build against. The normative text is [SPEC.md](SPEC.md); the machine-readable
schemas, vocabularies, conformance definitions, and executable examples live
alongside it.

## Isn't this just OKF?

No. The two specify different layers. Google's Open Knowledge Format (OKF,
v0.1, June 2026) standardizes the *carrier*, a Git tree of Markdown with
YAML frontmatter, and keeps it permissive: one required field, and consumers
are forbidden from rejecting bundles over unknown types, missing fields, or
broken cross-links. That permissiveness is correct for OKF's job, which is
portable descriptive knowledge: tables, metrics, runbooks. It is wrong for
RAC's job, which is prescriptive product decisions with accountability.

RAC writes the same carrier and specifies the semantic layer OKF explicitly
leaves to producers: a closed `status` lifecycle with supersession, a closed
typed-relationship vocabulary with referential integrity, explicit `id`-based
identity that survives file renames (OKF's path-as-identity does not), and
write-time enforcement — `rac validate` and `rac gate` reject broken links
and references to retired decisions in CI, before the knowledge lands. The
two compose rather than compete: `rac export --okf` emits a conformant OKF
bundle, so any OKF consumer can read a RAC corpus. The reverse direction (an
arbitrary OKF bundle becoming a validated RAC corpus) is not generally
possible. RAC is the strict layer; OKF is the permissive carrier beneath it.

## Why does the semantic layer need to be strict when OKF chose permissive?

Because descriptive knowledge degrades gracefully when stale, and
prescriptive knowledge does not. An agent reading a slightly outdated table
description writes slightly worse queries. An agent citing a superseded
architecture decision re-introduces the exact mistake the team already paid
to rule out. The failure modes are not symmetric, so the consumption rules
should not be either. Enforcement at write time — a merge gate that fails on
a reference to a retired decision — is the difference between a wiki and a
system of record.

## What is Lore, then?

Lore is the product surface: the MCP server identity, CLI branding, and
tooling built on RAC. This repository is implementation-neutral. Nothing in
the specification requires `rac-core`, Python, or MCP; everything normative
is checkable from the files alone.

## Can I implement this without rac-core?

Yes. That is what this repository is for. Start with
[SPEC.md](SPEC.md), then [`schema/`](schema/) for the machine-readable
frontmatter and per-type structural contracts, [`vocabulary/`](vocabulary/)
for the closed `status` and relationship enums,
[`conformance/`](conformance/) for what *conformant producer* and *conformant
consumer* mean, and [`examples/`](examples/) with its `manifest.json` as an
executable acceptance suite: every valid corpus must produce zero blocking
findings, every invalid case exactly its documented finding.

The specification defines two corpus levels — *conformant* (structural
validation clean) and *gated* (link integrity, no references to retired
decisions, review tiers 1–2 empty) — plus producer and consumer conformance.
If you build an implementation, open an issue here to have it listed.

## How stable is this?

This is v0.1.0, extracted from a validator with more than a thousand commits
and twenty-eight releases of dogfooding behind it. It is still pre-1.0, and
breaking changes are expected. The compatibility policy in
[SPEC.md §10](SPEC.md) governs what may change in minor versions (new enum
values, new advisory checks) and what requires a major version (removing or
re-defining values, changing tier assignments). Changes are tracked in the
[CHANGELOG](CHANGELOG.md) and governed by the process in
[CONTRIBUTING.md](CONTRIBUTING.md) and [GOVERNANCE.md](GOVERNANCE.md).
