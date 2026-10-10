# Plan-Review round 1 — fixer report

Every finding was checked against the code and the decision record, and every premise held, so all 23 are fixed.
`lyx loom validate-plan` exits 0 after the fixes; its one finding is the informational `create-new-unit` for card 17's new `roleguard_test.go`.

## Fixed

- **F1** (card 3, card 4): card 3 now keys every `freshRestartSteps(stepRun)` site of `Rebaseline` (the no-frame `overviewRefusal`, the begun batch's card-set change, both changed and done-changed branches) on `RebaselineDeps.Step`.
  It also re-signs `CheckBatchOrder` to take the step, with its callers `formBatches`, `ExecutionBatches` and `rebaselineBatches` and `sequence_test.go` on the card.
  Step-set rows now cover the mixed done-and-card-set case and an out-of-order follow-up card, and assert neither `lyx webster run` nor `lyx webster rebaseline` appears.
  Card 4 asserts both verbs are absent on every `ErrAutoRebaseline` row, including the out-of-order follow-up card.
- **F2** (card 14): adds `internal/loomcli#loomCLI.wire` and `wiring_test.go`; the loom's own `RunDeps` literal sets `ReadOnly`, and `TestWire_DefaultConfig` asserts it beside `RefMatcher`.
- **F3** (card 21): adds `Run.shellWaitActive`, `Runner.start` (sets started-fresh), `Runner.attach` (the one builder behind the Attach family, which leaves it unset), `heldTurnEnd` and `holdNotice` (the dead `tasks` field and task listing are dropped), `waitmark_test.go` and `hold_test.go`.
  The card states what the marker shows once nothing expires, and that `Run` drops `expiredShells`/`expiredLabels`.
- **F4** (card 19): adds `loomengine#LoadConfig` and its test.
  The exemption premise is corrected: `fabricengine#LoadConfig` (reached by the reader verbs) and `fabricengine#ReadOriginFor` (reached through `hubgeom.ResolveParent`) join the set with their tests.
  A new `TestReadOriginFor` pins the missing-lock text, since no test read it, and the fabricengine budget row rises.
  The card names which remaining `fabricengine` texts stay exempt and why.
- **F5** (card 11): the check scans `Card.Text`, reads no file, and `Card`'s `Text` field comment names the new reader (`internal/planparser#Card` moved from `Uses:` to `Edit:`).
- **F6** (card 2): creates the pure parser `commitsFromBatchCheck`, table-tested over literal batch-check output (commit/blob/tree lines, `missing`, malformed, empty) in a subtest beside the existing real-git rows; the open process-count claim is dropped.
- **F7** (card 16, formerly 17): the verify row moves to a new non-parallel `TestVerify_DropsStrandName` naming the variable, and the verifytree budget row rises.
  The gate row passes the variable through the gate process's own spawn environment, so it stays parallel.
- **F8**: cards 16 and 17 are swapped: the gate and verify env drop is now card 16, the guard card 17, with the Card Index and card 13's cross-reference updated.
- **F9** (card 4): creates `appendRebaselineWarning`; `finishMasterDone` writes the warning into `summary.md`'s `Plan rebaselined` section, and the test asserts it there.
- **F10** (cards 3, 4, 5): card 3 wraps a `restampBaseline` failure in the keyed transient clause and adds rows for the changed-file read and card-hash read transients.
  Card 4 adds the re-step clause to a `SaveState` or `changedPlanFiles` failure, with rows for a save and a restamp failure.
  Card 5 now credits card 4 for that text rather than claiming it already exists.
- **F11** (card 15): a nil `ReadOnly` skips the in-line route and keeps `needs_fresh`, with a row.
- **F12** (card 23): `TestRepositoryProbes` runs `nonMergeCommitsBetween` against real git (commit past start, merge-only range, absent start SHA); `gitwrap_test.go` joins the targets.
- **F13** (card 9): `TestBurlerRoundEntry_RubricStencil` gains the fixer-rubric row and `bouncerprompt_test.go` the seed and judge rows; both files join the targets.
- **F14** (cards 13, 17): card 13 creates `fabricengine#FabricReaderVerbs` as the one reader-name declaration.
  Card 17 lists it under `Uses:`, builds `readerVerb` from it, and its subcommand walk pins every name it returns as a guard reader.
- **F15** (card 4): the stuck-reason prefix is applied after the verify-gate demotion, every error `Run` returns after an accepted rebaseline opens with the warning, and a demoted-case row is added.
- **F16** (card 6): `TestEditedDoneCards` keeps only what card 7's gate rows do not pin (edited done card, begun-not-done edited card, unreadable file, nil state).
  The unedited, unbegun, no-record and no-hash cases are pinned once, in `TestNewPlanGate`.
  I kept the top-level test instead of folding it into an existing one: no existing test covers `DoneCards` or its neighbours (`begun_test.go` holds only `TestDispatchScope`), so `EditedDoneCards` is a new surface, and a new test for it is what `PATTERN-test-economy` allows.
- **F17** (card 23): a `TestRebaseline_CardSet` row asserts `Recoveries` and `RecoveryStartSHA` survive a rebaseline; `rebaseline_test.go` joins the targets.
- **F18** (cards 17, 18): `help_test.go` asserts `add`'s long help says it pushes, `TestConfigOverRealHub` gains a per-worktree write row under a webster strand name, and card 18 adds a claim row to `contracts/stencils/claims_test.go`.
- **F19** (cards 15, 17): each card gains a `Bound:` sentence (card 15: classifier shape plus the evidence rule; card 17: what a typed bypass `push` lets through).
- **F20** (card 23): a failed HEAD read returns before `Starter.Start`, counting and recording nothing, with a re-run way forward, a spec row `recovery start unread` and a test row.
- **F21** (cards 3, 6): both list `internal/websterengine/doc.go` and state the passage.
- **F22**: card 13 gains one accepted row per git form; card 11 gains at-cap, one-over and multibyte rows; card 16 builds the stripped environment once before the slot branches; card 21 asserts the end-of-run log in shuttleengine's rows.
  Card 4 drops the rows `TestRun_RefusesBeforeSpawn` already covers and adds the assertion that the recovery prompt renders the amendment.
- **F23**: card 19's and card 9's ImpactSummary tallies are gone, card 19's `hubgeom` test target and its false claim are dropped, and card 17 (formerly 16) no longer lists the unexported `configcli#hubWriteRefusal` under `Uses:`.

Cards 16 and 17 were swapped with a plain `mv` of the two card files, since a rename has no Edit/Write form; no file content was rewritten by a script.

## Disputed

None.

## Deferred

None.
