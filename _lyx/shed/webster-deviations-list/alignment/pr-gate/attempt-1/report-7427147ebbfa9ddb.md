---
verdict: APPROVED
---

The diff of pull request 668 at `575ac642d` against `main` matches the board entry's goal and the decision record's three decisions, with no change the goal does not call for.

## Against the goal and the decisions

- **D1, file and package coverage in `ScopeGuard`.**
  `internal/planglyph/scope.go` adds `symbolFilesCovered`, which skips member glyphs and member `plan:` handles through `planparser.RefMember` and covers a file equal to a file scope or directly in a directory scope, never a subdirectory.
  A Modified symbol passes every `Before` and `After` file, a Renamed pair falls back only when both `From.File` and `To.File` are covered, and Created and Deleted use their own `File`, as D1 fixes per delta shape.
  The exact-match and either-side rename checks are unchanged, the symbol-less file check is unchanged, and the guard stays informational.
  The doc comment states the new coverage, and `internal/planglyph/doc.go`'s check list follows it.
- **The `planparser` accessor.**
  `refMember` becomes the exported `RefMember` in `internal/planparser/targetdirs.go`, reading a nil plan as the default language, as the scope's In list names and `PATTERN-ref-shape-registry` requires.
- **D2, one rule in step 2.**
  `contracts/stencils/webster/webster-body-implementer.md` states the rule once in per-card step 2: member, file, package and rename coverage, `**Uses:**` adding none, the three sanctioned deviations with their bounds, and that the fork lists, compares and reports none and a deviation alone never makes the report `FAILED`.
  The guard-step clause now points at step 2's rule, and the prior-batch sentence is untouched.
- **D3, the plan spec.**
  `contracts/specs/loom-plan-spec.md` keeps the informational and `Uses:` facts, drops the `UnitPath`-mapped comparison and the flat-target-set sentence, and points at `planglyph.ScopeGuard` beside the `internal/websterengine` pointer.
- **Out of scope respected.**
  The diff leaves the fork's report shape, `record-batch`, `recover-batch`, the digest, the Master template, `webster-body-verify-fix.md`, finding F12 and the `weaver` note alone.
- **Tests.**
  `ScopeGuard`'s tests fold into one table covering every acceptance criterion on the guard, and `TestRefMember` covers the new accessor; no test pins stencil prose.

## Open risks carried over

- The Discussion-Review carry-over lists three round-1 findings, on the deciding file for a moved symbol, the `Uses:` bullet against D1 and the missing `planparser` export.
  The decision record as it stands settles all three, in D1's per-shape file rule, D2's `Uses:` bullet and the scope's `internal/planparser` line, and the diff follows each.
- The open risk on `card-own-entry-edit` is closed: it landed on `main` as `c7ea67f83`, which does not touch `contracts/specs/loom-plan-spec.md`, and the branch's merge of `main` at `1d1a8fef4` went through Merge-Review with no finding.

## Overlap

- **Peer `creel-stage1-fixes`** (row 3, at Discussion-Write, own row 22): no changed paths, no decision record and no plan yet, and its subject, creel, shuttle and reed, does not touch `planglyph`, `planparser` or the Webster stencils.
  No overlap.
- **Parent commits:** `lyx loom peers` lists none; the branch already contains `main`'s head.
- **Shared bookkeeping:** `cmd/lyx/testdata/test-budget.yaml` raises `internal/planparser` from 95 to 96; it does not escalate.

## Superseded

No board entry is made obsolete.
`stencil-intent-audit` covers every stencil and keeps F12, and `weaver` decides its own implementer's deviations.
