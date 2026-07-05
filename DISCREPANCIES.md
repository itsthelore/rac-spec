# DISCREPANCIES — extraction working notes for human review

Scratch file from the v0.1.0 spec extraction. Each entry is a place where the
validator code, existing documentation, the extraction brief, or this
specification disagree. Per the extraction ground rules, none of these were
silently resolved: **the code's behavior is what SPEC.md specifies**, and the
disagreements are recorded here for human sign-off. Resolve (or consciously
accept) each entry, then delete this file before announcing v0.1.0.

---

## D1 — "Four severity tiers" vs. six review priorities

**Brief said:** rac-core classifies every finding into four severity tiers
(1 blocking … 4 advisory).
**Code says:** `rac.services.review` defines **six** priority levels:
1 invalid artifact, 2 broken relationship, 3 unknown artifact, 4 missing
recommended, plus advisory-only 5 `stale-corpus` (v0.13.3, opt-in) and
6 `suspect-artifact` (drift). Priorities 1–2 fail; 3–6 never do — so the
"only tiers 1 and 2 fail CI" claim holds.
**Spec does:** SPEC.md §9.2 specifies all six tiers, noting 5–6 as advisory
extensions of the original four-tier model.
**Also note:** the tier taxonomy applies to *review* findings. Structural
validation findings carry `error`/`warning`/`info` intrinsic severity and a
`blocking`/`advisory` enforcement class; they are not individually assigned
tiers 1–4. The normative per-check mapping in SPEC.md §9.3 is therefore
check → (intrinsic severity, default enforcement), which is exactly what the
code implements. **Sign-off needed:** none (faithful to code), but the brief's
four-tier framing should not be reintroduced in marketing copy.

## D2 — Unknown relationship-section names are silently inert

**Spec intent (brief):** an unknown relationship type should be a tier 1/2
finding (conformance-breaking).
**Code says:** two different behaviors:
- A *vocabulary* section on a type that does not declare it →
  `relationship-edge-unsupported` (warning severity, **blocking by default**
  in `rac gate`; fails `rac relationships --validate`). Conformance-breaking. ✔
- A section name *outside* the vocabulary entirely (e.g.
  `## Related Widgets`) → an ordinary body section: no edge, **no finding**. ✘
**Spec does:** §9.6 item 2 requires consumers to reject populated
vocabulary-edge violations (matching the code); out-of-vocabulary section
names remain free content per §11 (also matching the code — they are
indistinguishable from ordinary prose sections). **Sign-off needed:** decide
whether a future minor/major version should add a heuristic check (e.g. flag
`## Related *` headings outside the vocabulary). Ratifying v0.1.0 as-is is
consistent with the code.

## D3 — Frontmatter `relationships` kinds are not vocabulary-checked

**Code says:** `parse_frontmatter` validates the `relationships` field's
*shape* only (kind → list of strings); any kind string is accepted. The field
is reserved and unconsumed (`core/metadata.py`).
**Spec does:** §8.6 marks the field reserved; the frontmatter schema mirrors
shape-only validation with a `$comment`. **Sign-off needed:** when the field
is activated, unknown kinds must become blocking findings to honor the §9.6
strictness intent — record that as a planned minor/major change.

## D4 — `relationship-target-superseded` and `relationship-self-reference` are warning-severity

**Brief said:** unknown status → tier 1; retired-target references
conformance-breaking (implying error-tier).
**Code says:** both checks carry **warning** intrinsic severity
(`RELATIONSHIP_SEVERITY`), but every relationship finding fails
`rac relationships --validate` and defaults to **blocking** in `rac gate` —
so they are conformance-breaking despite the warning annotation level.
**Spec does:** §9.1 makes intrinsic severity and enforcement class explicitly
independent; §9.3 records warning/blocking for these codes. Faithful to code;
no change needed, but reviewers should not "fix" the warning severity to
error without a major-version bump (tier reassignment rule, §10.2).

## D5 — Corpus-level spec-version declaration does not exist in rac-core

**Spec says (deliberate, authorized addition):** SPEC.md §10.1 — every corpus
MUST declare `rac_spec: "0.1"` in `.rac/config.yaml`; §10.3 — consumers MUST
refuse newer-versioned corpora with a two-version diagnostic; §9.6 item 4 and
the §10.3 error-message rule depend on it.
**Code says:** `.rac/config.yaml` exists and is the right home (the engine
already reads `repository_key`, `ticketing`, `validation`, `enforcement`,
`audit` from it; unknown keys are ignored, so declaring `rac_spec` today is
harmless) — but **no code reads it**. The only version the engine reads is
the per-artifact frontmatter `schema_version` (supported: {1};
`unsupported-schema-version` error otherwise — which does implement the
strict-refusal semantics, per artifact).
**Action required before announcing v0.1.0 (do not skip):** either implement
`rac_spec` reading + newer-version refusal in rac-core, or demote §10.1's
producer MUST (and the §9.6.4 corpus clause) to SHOULD for v0.1.0. SPEC.md
§10.1 carries an inline implementation-status note so the requirement cannot
ship silently.

## D6 — OKF key preservation is SHOULD, not MUST

**Brief said:** RAC's extra frontmatter keys ride as OKF producer-defined
keys, "which OKF consumers MUST preserve per OKF's consumer rules."
**OKF spec text says (verified against okf/SPEC.md, 2026-07-05):** consumers
"SHOULD preserve unknown keys when round-tripping and SHOULD NOT reject
documents with unrecognized fields" — SHOULD, not MUST. The MUST-NOT-reject
list (OKF §9) does cover unknown keys, so RAC bundles are safe from
rejection; preservation is only SHOULD-strength.
**Spec does:** SPEC.md §5 item 2 says OKF consumers "are directed to preserve
and not reject" — accurate without overclaiming. No action needed; recorded
so nobody "corrects" it back to MUST.

## D7 — Producer-defined frontmatter keys vs. the closed envelope

**Spec intent (brief, §11):** producer-defined frontmatter keys are permitted
and preserved (to match OKF losslessness).
**Code says:** the reference validator **rejects** unknown frontmatter keys
(`invalid-metadata-field`, error) — ADR-025's closed envelope.
**Spec does:** §11 states the extension principle but flags that the
reference validator currently rejects unknown keys, and directs producers to
`tags` until an extension-key namespace exists. **Sign-off needed:** decide
the v0.x plan — e.g. reserve an `x_`/`ext.` namespace (minor version) — or
drop the producer-defined-keys promise from §11 and extend via `tags` only.
The current §11 text is honest about both; a cleaner resolution should land
before v0.2.0.

## D8 — OKF's own reference parser vs. the OKF spec text

**Known ecosystem discrepancy (from the brief; not independently
reproducible offline):** Google's reference parser has been observed to
require `type`, `title`, `description`, `timestamp`, though the OKF spec
text requires only `type`. RAC's export targets the spec text and emits no
`title`/`description` frontmatter. Recorded as a compatibility note in
`conformance/conformance.md`. **Sign-off:** accept targeting the spec text
(recommended), or extend `rac export --okf` to also emit `title`/
`description` for parser compatibility (an rac-core change, out of scope
here).

## D9 — "IDs survive file moves" attributed to the wrong release

**Brief said:** the migration behavior comes from the 2026.06.5 "rename
release."
**History says:** 2026.06.5 renamed the *PyPI package*
(`requirements-as-code` → `rac-core`, ADR-092). Path-independent identity
dates from ADR-026 / v0.7.11 (opaque frontmatter IDs + alias resolution);
the artifact-id refactor tooling (`rac rename`) landed in v0.21.18. The
normative property itself is real and specified in §6.4. No spec change
needed; noted so the release citation is not copied into announcements.

## D10 — Review `broken-relationship` coverage vs. gate relationship findings

**Observation (informational):** review tier 2 (`broken-relationship`)
derives from portfolio resolution issues (not-found / ambiguous / self);
range, cycle, status-consistency, edge-legality, and duplicate-identifier
findings reach the gate through the *relationships* source (all blocking by
default) rather than through review tiers. The tier table and the check
table in §9 reflect this split faithfully. No action; recorded because the
brief's "tier 2 = broken relationships" shorthand under-describes which
service carries which check.
