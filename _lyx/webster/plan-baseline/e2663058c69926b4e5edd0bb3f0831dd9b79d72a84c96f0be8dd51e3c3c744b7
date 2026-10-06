# Card 4 — parsers-prune

**Edit:**
- `internal/planparser#`
- `internal/discussionparser#`
- `internal/summaryparser#`
- `internal/shedbuild#`
- `internal/mergeresolve#`

**Intent:**
Run the prune procedure (Shared Decisions, `prune-procedure`) on the sole-parser packages' test files (`PATTERN-sole-parsers`) and `internal/mergeresolve`.

`internal/planparser` carries the largest candidate set of the whole task.
Parser tests that feed one input shape and check one finding or field are the main fold leads: one table per behavior (per finding code, per field, per ref kind), every input and expected value kept as a row.
A test that pins a finding's code, message or location is a unique assertion under Decision 2 unless its covering test checks the same.
`PATTERN-ref-shape-registry` tests on `planparser`'s kind-policy ledger keep one row per kind.

**ImpactSummary:** Test files of five parser packages only; no production change.

**Commit:** 4: Prune parser tests
