# Examples

The executable conformance surface for the RAC (Requirements as Code)
specification. `manifest.json` is the machine-readable index: each entry names
the command to run against the reference implementation, and the finding
codes (with intrinsic severity and default enforcement class) the case must
produce.

- `minimal-corpus/` — SPEC.md Appendix A verbatim: the smallest corpus that
  validates with zero blocking findings.
- `valid/` — one minimal valid artifact per type, cross-linked so the
  relationship checks also pass.
- `invalid/` — one case per major rule. Single files exercise structural
  rules; `corpus-*` directories exercise corpus-level relationship rules.
  Every case carries an HTML-comment annotation naming its expected finding.

The example corpora are independent: do **not** validate `examples/` as one
tree. Invalid cases are invalid by design, and identifiers are only unique
within their own corpus.
