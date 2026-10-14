---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: _lyx/plan/02-rename-batch-boundary.md:55
    summary: Card 2 plans two test rows that pin no behavior an existing row leaves uncovered, and the TestExecutionBatches sentence describing one of them is garbled
    fix: In card 2, drop the Edit targets `internal/batcher#TestIdentityBatcher_Batch` and `internal/websterengine_test#TestExecutionBatches` and the Uses entries `internal/websterengine_test#partitionPlan` and `internal/websterengine_test#batchIDs`. Delete the Intent paragraph starting "Add a row to `TestIdentityBatcher_Batch`" and the paragraph starting "`TestExecutionBatches` gains an optional `plan` field". Replace them with one sentence saying the identity batchifier and `ExecutionBatches`' recorded-partition path are unchanged and keep their existing tests (`TestIdentityBatcher_Batch`'s threeCards row and `TestExecutionBatches`' row "a recorded partition is kept when the size source changes between calls"). `lyx loom validate-plan --overlay` with these drops applied reports nothing the `{}` baseline does not.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/06-guard-step.md:83
    summary: The unreadable-directory rows of TestRecoverSpawnOrAttach (card 6) and TestScanGuardTests (card 5) name no root go.mod, so card 5's no-go.mod early return can keep the walk and its refusal from being reached
    fix: "In card 6's `TestRecoverSpawnOrAttach` bullet, change \"and a row with an unreadable directory\" to \"and a row whose worktree holds a root `go.mod` and an unreadable directory\". In card 5's `TestScanGuardTests` list (line 45), change \"an unreadable directory (mode `0o000`, …)\" to \"a root `go.mod` and an unreadable directory (mode `0o000`, …)\". Change no target."
  - id: F3
    severity: LOW
    class: decision
    location: _lyx/plan/05-guard-scan.md:32
    summary: An unreadable `_test.go` file inside a collected directory reaches the fork as a GuardScanError prompt line, not as D3's read-error refusal, and no card settles which one it should be
    fix: "Settle it in card 5's Intent. Recommended: the `scanGuardTests` walk opens and closes each `_test.go` file it collects, and returns a failed open as the walk read error (`webster: walk the worktree for guard tests: <path>: <error>`). Add a `TestScanGuardTests` row: a root `go.mod` and a `_test.go` file with mode `0o000`, skipped on Windows and as root and restored in `t.Cleanup`, gives an error naming that file and no `scanErr`. This leaves impactset's `guardNamesInFile`, and so the round gate's error policy, unchanged. Alternatively, state in card 5's Intent that an unreadable test file deliberately stays a `GuardScanError` prompt line, and name the D3 bound it falls under."
---

The plan carries D1 to D4 and every Constraint.
Card order holds, every targeted package is under `## verify:` in the template's order, and every `Edit` and `Delete` card's `ImpactSummary` states a real blast-radius conclusion.
The findings below are all LOW.

### [LOW] Card 2 plans redundant test rows

**Location:** `_lyx/plan/02-rename-batch-boundary.md:55` (the TestIdentityBatcher_Batch row) and `:57` (the TestExecutionBatches row)

**Issue:**
`identityBatcher.Batch` never reads a card's target groups, and card 2 does not touch `identity.go`.
A row of tender-batch-07-shaped cards therefore pins the same one-batch-per-card-in-order behavior as the existing `threeCards` row of `TestIdentityBatcher_Batch`.

`ExecutionBatches` (`internal/websterengine/partition.go:60`) never calls the active batchifier when `state.json` holds a partition; it maps the partition and runs `CheckBatchOrder`.
The existing row "a recorded partition is kept when the size source changes between calls" already pins that a recorded partition survives a batchifier that would group differently.
The planned row adds nothing to that, all the more because its `Budget: 1` makes every multi-card segment infeasible even without the boundary rule.
The sentence describing the row is also garbled: it repeats "a two-card plan (`Language: "go"`, card 1 slug `a` …)" twice, joined by "and a row over".
`PATTERN-test-economy` covers both rows.

### [LOW] Unreadable-directory rows don't name the root go.mod the walk needs

**Location:** `_lyx/plan/06-guard-step.md:83`, `_lyx/plan/05-guard-scan.md:45`

**Issue:**
Card 5 has `scanGuardTests` return the zero value and no error when the worktree root has no `go.mod` (`05-guard-scan.md:26`), before it walks anything.
Card 6's `TestBeginBatch_Refusals` row names "a root `go.mod` and a directory chmodded `0o000`".
The `TestRecoverSpawnOrAttach` unreadable row says only "a row with an unreadable directory, restored and skipped the same way", and card 5's own unreadable row also names no `go.mod`.
As written, those inputs are rejected by the no-`go.mod` return upstream of the walk, so the recover-batch refusal is not reached by the input the card names.
An implementer would notice the failure and add a `go.mod`, but the card should name the input that reaches the refusal.

### [LOW] An unreadable test file lands as a scan error, not a refusal

**Location:** `_lyx/plan/05-guard-scan.md:32`; also `_lyx/plan/03-export-guard-scan.md:22`

**Issue:**
`guardNamesInFile` (`internal/impactset/guards.go`) calls `parser.ParseFile` with nil source, so the parser reads the file itself.
A failed read comes back as `&GuardScanError{message: "impactset: parse …"}`.
Card 5 keeps every `GuardScanError` in `scanErr`, so a `_test.go` file that cannot be read renders card 4's scan-error prompt line and begin-batch goes on.
D3 says a read error is not a test-file fault and refuses begin-batch and recover-batch.
Card 5 applies that refusal only to directories and entries the walk cannot read.
Card 3 edits `guardNamesInFile` but keeps the classification, and reclassifying there would change the impactset round gate's error policy, which D3 says stays as it is (`internal/loomshed/verifygate.go` turns a `GuardScanError` into findings and any other error into a gate error).
The author has to choose where this case lands.

## Rejected

- Card order across 3 to 6: card 5 uses `FindGuardTests`, created by card 3's rename new side, and card 6 uses `MaxCommandLength` (card 3), the three fragment vars (card 4), `guardTests`, `scanGuardTests` and `writeGuardFixture` (card 5), and `RefCovers` (card 1). Every dependency points to an earlier card. Not a defect.
- Cards 1 and 2, member glyphs and re-sign arrows: card 1 creates handles and edits only a non-Go file. Card 2 lists `costBatcher.Batch` and each test it changes as a member glyph, and its signatures don't change. Card 6 carries the arrow on `cardGateCommand` and `renderCardGates`, the two members whose signatures change. Not a defect.
- Card 6 granularity: `guardTests.step` and `guardNotice` are private helpers serving the one card-gate render surface. The `cardGateCommand` and `renderCardGates` signature changes force the `BeginBatch` and `recoverSpawn` call-site edits into the same compile unit. The rubric bundles a helper serving one surface into that surface's card. Not a defect.
- Card 6 refusal and fallback reachability: the begin-batch unreadable-path row names a root `go.mod` and a `0o000` directory. The fake index's `ValidateDispatch` and `batchCardHashes`, which reads the plan dir, run upstream and don't reject it. The misplaced-marker and over-long rows each name an input and a test (`TestBeginBatch_PromptFile`, `TestCardGateCommand`). The recover-batch row is F2.
- Card 2's `renameBoundaries` check of only `boundaries[begin]`: it is correct because of the existing early `break`. As `begin` falls, every shorter suffix ending at `end` was already found feasible, which the card's extended break comment states.
- Card 2's new `TestRenameBoundary_BatchesCheckAndGateTheirOwnTree`: it pins the decision record's tender-batch-07 acceptance criterion end to end, which no existing test composes. Kept.
- Test budget: `internal/websterengine` goes from 200 to 202 against a row of 202 (cards 2 and 5). `internal/planparser` goes to 94, and card 1 raises its row. `internal/batcher` gains no top-level test. Not a defect.
- Attack surface: card 4's Intent states what D4's implementer-body paragraph lets a fork change (a file outside the card's targets that the guard step's failure names, a pin only when the pin itself must move) and its bound (every other deviation rule stands). Card 6 states that a scan error or an over-long step drops only the guard step. Card 4's fragments say the plan-level verify still runs the guard tests. Not a defect.
- `## verify:`: `go test ./...` and `go test -tags integration ./...` cover every targeted package, including `cmd/lyx` for `test-budget.yaml`. The only nested module, `internal/planglyph/testdata/glyphchain`, is not targeted. The order matches the template. Not a defect.
- Card 6's statement that `scanGuardTests` runs "before their first mutation" in `BeginBatch` is loose: `ValidateDispatch` may already have rewritten handles and `restampFingerprint` persisted state by then. The placement still matches the existing transient refusal of `batchCardHashes`, so no finding is raised.
