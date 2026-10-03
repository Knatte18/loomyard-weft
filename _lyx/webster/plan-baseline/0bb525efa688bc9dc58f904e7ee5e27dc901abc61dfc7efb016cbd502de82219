# Card 23 — claims-table

**Create:**
- `contracts/stencils/claims_test.go`

**Edit:**
- `contracts/stencils/rubric_test.go`
- `contracts/stencils/priorplantemplate_test.go`
- `contracts/stencils/focusdirective_test.go`
- `contracts/stencils/plantemplate_test.go`
- `contracts/stencils/discussiontemplate_test.go`

**Intent:** Create the one claims table for raw-stencil wording (D12) and move `contracts/stencils`' own wording checks into it.
`claims_test.go` holds a table keyed by stencil file, each claim a `must` or `mustNot` phrase with a `why`, and one table-driven test that reads each stencil's raw embedded text once and checks its claims, naming file, phrase and `why` on failure.
A claim may also name a section heading, which limits its check to that heading's section, up to the next `## ` heading, and a claim naming a heading the stencil lacks fails;
a check scoped to one section today stays scoped, so it is neither weakened into a whole-file `must` nor strengthened into a whole-file `mustNot`.
A claim's `why` keeps its provenance (crucible round, finding, sabotage proof);
a claim that cites one is protected (D21).
The raw `strings.Contains` checks in the listed files move into the table;
claims that pin generic wording rather than a behaviour are dropped, each weighed against its `why` and listed with its reason in the commit body.
Left in place, per D12: marker-set and `Fill`/`FillOptional` round-trips, Go-contract co-versioning checks, the rubric marker allowlist and `citation_enforcement_test.go`.
What this lets through, bounded per D12: a dropped generic-wording phrase can change unnoticed;
only claims without cited provenance are dropped, and each is itemised.

**ImpactSummary:** `contracts/stencils` tests only; wording assertions move into one table, other checks unchanged.
