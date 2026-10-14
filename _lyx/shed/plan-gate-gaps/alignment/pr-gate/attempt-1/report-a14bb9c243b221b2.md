---
verdict: APPROVED
---

## Alignment at `pr-gate`: pull request 614 at `23b6dc289` against `main`

The diff matches the board entry's three scope points and the decision record's D1–D4, and it adds nothing they rule out.

- D1: `internal/batcher/boundary.go` adds one rule, `renameBoundaries`.
  The cost batchifier calls it, treating a segment that holds a covering pair as infeasible, and a one-card segment stays feasible.
  The covers check is `planparser.RefCovers`/`CardCovers` in `targetdirs.go`, so the ref-shape logic stays in `planparser`.
  `DoneChecks`, `BindHandles`, `DetectDrift`, the scope guard and `gateScopeOf` are unchanged, as D1's rationale says.
  The identity batchifier is untouched.
- D2: `internal/websterengine/guardstep.go` walks the root module without spawning a process, skipping `testdata`, `vendor`, `_`/`.` directories and nested `go.mod` subtrees.
  The scan goes through the now-exported `impactset.FindGuardTests`.
  `cardGateCommand` appends one `lyx gate test --tags integration … -- -run '^(…)$'` step before the two lints.
  A guard directory drops out once the cards up to that card have emptied it.
  begin-batch and recover-batch both render the step.
- D3: a `GuardScanError` renders the scan-error fragment, plus the fix fragment when the failing file lies in a package a card targets.
  An over-long step renders the too-long fragment.
  A walk read error refuses before any mutation, naming the path and saying to re-run the verb.
  `refusal-spec.md` gains its row, and tests reach it in both verbs.
  impactset's round gate keeps its own error policy.
- D4: `webster-body-implementer.md` carries the instruction to fix outside the card under the card's commit and to move a pin only when the pin itself must move.
  `claims_test.go` pins that wording.
- Docs: `batcher/doc.go`, `websterengine/doc.go`, `impactset/doc.go`, `loom-plan-spec.md` (batching and per-card gate) and `refusal-spec.md` all land in the PR, as Scope requires.
- Carried-over open risks from Discussion-Review are addressed in the code:
  - `d3-walk-read-error-no-refusal-spec-row`: the row and its tests landed.
  - `d2-drop-parenthetical-all-files`: the drop is computed per file through `RefCovers`, and a Create adds files back.
  - `d4-fork-may-loosen-guard`: the stencil limits pin changes to a pin that must move.
- Every acceptance criterion has a named test in `cost_test.go`, `targetdirs_test.go`, `cardgate_test.go`, `guardstep_test.go`, `beginbatch_test.go` and `recoverbatch_test.go`.

## Overlaps

`lyx loom peers` reports `own_row` 22 and no parent commits; `main`'s HEAD `7e284702b` is this branch's merge base.

- `claimed-scope-edit` (row 17, Merge-In) also changes these files:
  - `contracts/specs/refusal-spec.md`;
  - `contracts/stencils/claims_test.go`;
  - `contracts/stencils/stencils.go`;
  - `cmd/lyx/testdata/test-budget.yaml`, a shared bookkeeping file.
  The overlap is text only, with different rows, claims and registry entries.
  Its decisions cover darn's decision record and PR-Align, not webster's batching or card gates, so no decision conflicts.
  Landing order: this run (row 22) is further along and lands first; `claimed-scope-edit` takes these changes at Merge-In.
- `twb-fx-ia`, `twb-fx-ib`, `twb-fx-ic`, `twb-fx-id`, `twb-fx-wa`, `twb-fx-wb`, `twb-fx-wc`, `twb-fx-wd` (row 8): no changed paths, no overlap.

## Superseded

None.
The board entries this task absorbed (`webster-glyph-rename`, `plan-pinned-tests`) are no longer on the board.
