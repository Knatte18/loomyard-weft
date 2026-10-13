---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/websterengine/classify.go:4
    summary: File headers in classify.go and digest.go still call the deviation list the fork's own, though the scope guard now alone fills it
    fix: In internal/websterengine/classify.go, change the header sentence ending "beyond the fork's own informational deviation list." to end "beyond the scope guard's informational deviation list." In internal/websterengine/digest.go, change the header phrase "webster's Digest carries the fork-return facts (status, head SHA, informational deviation list)" so it reads "webster's Digest carries the fork-return facts (status, head SHA) and the scope guard's informational deviation list".
---

### [LOW] Stale file headers still attribute the deviation list to the fork

**Location:** `internal/websterengine/classify.go:4`, `internal/websterengine/digest.go:5`
**Issue:** Card 7 moved the digest's `deviations` fill from the fork's report to the scope guard, and updated `Report`, `distill`, `Digest.Deviations`, `postBatchChecks` and the package `doc.go` to say so.
Two file headers in the same package still describe the list as the fork's own: `classify.go` says "the fork's own informational deviation list" and `digest.go` lists the deviation list among "the fork-return facts".
Both are now false, since the fork returns `status` and `head_sha` only.

## Verified without findings

- Cards 1–2: `Relaunch` calls `Admit` after every check, including `CheckGotoTarget`, and skips the `Target` move only when `inEffect` is true.
  `admitRelaunch` is the one check both consumes call; its refusal texts are the old ones.
  Its in-effect branch requires an unmoved-or-in-effect stop key, a paused run at the target, only goto entries since the spawn and the last naming the target.
  A pause condition now present that the spawn did not record refuses.
  `TestRelaunch` and the consume tables cover the in-effect, pause and moved-elsewhere rows through the real `Relaunch`.
- Card 3: the arbiter reads the verify record only for a blocked Merge-In or Publish stop, reads HEAD only when the record names a target, and refuses only when `battenshed.RecordMatchesHalt` holds.
  Every other blocked stop stays eligible.
  No `PublishFailureTarget` or `PublishHaltNamesTarget` reference remains outside the plan files.
  The refusal-spec row and the `doc.go` sentence match.
- Card 4: `row` comes from the board status via `recipeRow` over the embedded recipe's rows, and `own_row` appears only from a task pair.
  The alignment stencil's landing-order section reads `row` against `own_row`, names ties and unknown order, and restates no recipe row.
- Cards 6–8: `ScopeGuard` sets `File`, and the file-level pass is file-granular with `RefScope` beside `RefFile` in `planparser`.
  `postBatchChecks` returns a deduplicated, sorted list that never joins the warnings; `RecordBatch` and `PersistRecoveryTerminal` store it.
  `Report.Deviations` stays with `omitempty` and is never read, and the digest's JSON name and `omitempty` are unchanged.
  The implementer stencil asks for `status` and `head_sha` only, the Master stencil reads the list from the digest, and the digest line reads `scope-guard deviations=`.
- Per-card mechanical checks: every group in the cards is an Edit or Create, with no Rename or Delete group, so no `assert-no-callers` or rename script applies.
  The test-budget row for `internal/planglyph` rose by one for the one new top-level test.
- Test economy: each new test row pins a behavior no existing test covers; no helper has its own test beside a public-surface one.
- `lyx gate test` over `internal/loomcli`, `internal/shedengine`, `internal/planglyph`, `internal/planparser`, `internal/websterengine`, `contracts/stencils`, `internal/alignment` and `cmd/lyx` passes.
