# RAC (Requirements as Code) — the `status` vocabulary

Normative reference: [SPEC.md §7](../SPEC.md). The enums below are closed:
a present value outside its type's set is the blocking finding
`invalid-<type>-status`. A missing `## Status` section is always legal —
status is optional, validated-if-present. The value is the first non-empty
line of the `## Status` section body, matched case-insensitively.

## Values

| Type | Value | Meaning | Live / retired |
| --- | --- | --- | --- |
| requirement, decision, design | `Proposed` | drafted, not yet ratified | live |
| requirement, decision, design | `Accepted` | ratified; current knowledge | live |
| requirement, decision, design | `Superseded` | replaced by a newer artifact | **retired** |
| requirement, decision, design | `Deprecated` | withdrawn without a replacement | **retired** |
| roadmap | `Planned` | intent the team still holds | live |
| roadmap | `Achieved` | intent realized; a valid historical record (live *terminal* state) | live |
| roadmap | `Superseded` | intent replaced by another roadmap | **retired** |
| roadmap | `Abandoned` | intent dropped | **retired** |
| prompt | `Active` | in use | live |
| prompt | `Deprecated` | withdrawn | **retired** |

Status is knowledge currency — is this still what we believe? — never work or
delivery state. Per-milestone progress, ticket state, and sprint tracking are
out of scope by design.

## Legal transitions

The validator enforces **no transition graph**: any enum value may be written
at any time, and history lives in version control (a transition is a reviewed
diff). What is enforced is the consequence of the live/retired partition:

- A live artifact referencing a retired artifact through any edge except
  `supersedes` is `relationship-target-superseded` (blocking by default).
- References *from* retired artifacts are exempt (historical chains stay
  intact).
- A conformant consumer must refuse to present a retired artifact as current
  (SPEC.md §9.6).

The conventional lifecycles, for orientation (not enforced):

```text
requirement / decision / design:  Proposed → Accepted → Superseded | Deprecated
roadmap:                          Planned → Achieved | Superseded | Abandoned
prompt:                           Active → Deprecated
```

## Adjacent constrained enums

Two further closed enums ride the same mechanism (SPEC.md §7.1):

| Type | Section | Values | Unknown-value finding |
| --- | --- | --- | --- |
| decision | `## Category` | Architecture, Product, Process, Technical, Other | `invalid-decision-category` |
| roadmap | `## Horizon` | `now`, `next`, `later`, or `Q[1-4] YYYY` | `invalid-roadmap-horizon` |

Adding a value to any enum on this page is a **minor** spec version; removing
or changing the meaning of one is a **major** version (SPEC.md §10.2).
