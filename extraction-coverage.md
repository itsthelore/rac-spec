# Extraction coverage report

Working artifact of the v0.1.0 extraction (Phase 5.5); delete together with
`DISCREPANCIES.md` before announcing v0.1.0.

## Inventory → spec traceability

All 18 top-level keys of `extraction-inventory.json` are cited from SPEC.md
via `<!-- inv: <key> -->` source comments (mechanically checked; zero missing,
zero stale citations):

| Inventory key | Specified in |
| --- | --- |
| `artifact_types` | SPEC.md §6.1; schemas |
| `classification` | SPEC.md §6.1 |
| `file_format` | SPEC.md §6.2 |
| `frontmatter` | SPEC.md §6.3, §8.6; `schema/frontmatter.schema.json` |
| `id_rules` | SPEC.md §6.4; frontmatter schema pattern |
| `sections_per_type` | SPEC.md §6.6; `schema/artifact.schema.json` |
| `status_enums` | SPEC.md §7, §7.1; `vocabulary/status.md`; artifact schema |
| `relationship_types` | SPEC.md §8; `vocabulary/relationships.md` |
| `requirement_line_grammar` | SPEC.md §6.7; artifact schema |
| `severity_model` | SPEC.md §9.1, §9.2, §9.5 |
| `checks` | SPEC.md §9.3 (normative table) |
| `gate_semantics` | SPEC.md §9.4 |
| `exit_codes` | SPEC.md §9.4 |
| `spec_version_declaration` | SPEC.md §10.1 (+ DISCREPANCIES D5) |
| `reserved_structure` | SPEC.md §5, §6.2, §6.5 |
| `okf_composition` | SPEC.md §5; `conformance/conformance.md` |
| `limits` | SPEC.md §6.3 (envelope caps); parser-robustness checks in §9.3 |
| `ticketing_providers` | SPEC.md §8.5 |

## Validator behavior deliberately NOT specified (out of scope, with reasons)

- **Classification scoring internals** beyond the 0.5 threshold and the
  1 / 0.5 weighting (§6.1): tie-breaking by registry order is an
  implementation detail with no observable conformance effect on valid
  corpora.
- **Severity overrides / enforcement-policy YAML syntax** (`validation:`,
  `enforcement:` stanzas): §9.5 specifies the semantics (committed,
  deterministic, precedence `off` > `blocking` > `advisory` > default) but
  not the YAML key layout, which is CLI-contract territory owned by the
  reference implementation's docs. Candidate for promotion to normative in
  v0.2.
- **Robustness cap values** (1 MiB file cap, 64 KiB frontmatter, depth 32,
  256 KiB field, 50 k lines): §6.3 cites the envelope caps; the exact
  file/field caps are quality-of-implementation bounds, and the file cap is
  environment-overridable. The associated finding codes ARE specified
  (§9.3).
- **Review advisory mechanics for tiers 5–6** (stale-after window default of
  14 days, drift detection algorithm): git-derived, opt-in, advisory-only —
  never affect conformance (§9.2 rows only).
- **`rac watchkeeper`, `rac doctor`, search, MCP tool surface, exports other
  than OKF**: consumers/reporting layers, not corpus semantics. The doctor's
  gating subset is the same validation + relationship errors already
  specified.
- **Ticket-format regexes per provider**: §8.5 names the provider vocabulary
  and the finding; the per-provider key grammars are recorded in the
  inventory (`ticketing_providers.formats`) and can be promoted to the spec
  when a second implementation needs them.

## Phase 5 verification results (2026-07-05)

1. **Examples vs reference validator** — all 2 valid corpora pass
   `validate` + `relationships --validate` + `gate` with exit 0 / zero
   blocking; all 16 invalid cases fail with exactly the documented code at
   the documented intrinsic severity. (18/18 checks green.)
2. **Tier-mapping three-way cross-check** — validator source
   (`RELATIONSHIP_SEVERITY`, gate defaults), real `rac gate --sarif` output
   (warning-severity blocking finding renders SARIF `warning` while exiting
   1), and the GitHub Actions PR gate (fails exactly on non-zero exit) all
   agree: blocking findings and only blocking findings fail CI.
3. **OKF round-trip** — `rac export rac/ --okf` over the dogfood corpus: 389
   typed artifacts + generated `index.md` + `log.md`; every artifact carries
   a non-empty mapped OKF `type` and an `id`; zero violations. OKF's own
   validator is not available offline; the known Google-reference-parser
   discrepancy is recorded in `conformance/conformance.md`.
4. **Version declaration** — confirmed the engine reads no corpus-level
   spec-version key (`rac_spec` absent from `src/rac/`); see
   DISCREPANCIES.md D5 and SPEC.md §10.1's implementation-status note.
