---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/03-teardown-settles-board-claim.md:31
    summary: the landed read fails on a gone task worktree, so the gone-pair teardown route never settles the claim, against D10's every-route clause
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/03-teardown-settles-board-claim.md:45
    summary: the production childLanded and battenRunState substrates are reached by no planned test, so the fabric-remove clear, done and held outcomes are pinned only through fakes
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/13-batten-admits-darn.md:20
    summary: card 13 states the darn-seed bound as batten's admit set alone, while card 12 names the hand route through lyx shed seed as well
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/plan/06-verify-gate-told-command.md:40
    summary: the whole-diff gate form leaves a merge-base read error unstated and untested, so it could become a skipped comment lint
---

### [MEDIUM] The gone-worktree teardown route never settles the claim

**Location:** `_lyx/plan/03-teardown-settles-board-claim.md:31`

**Issue:** `RemovePair` reads `landed` through `childLanded` before the removal.
`childLanded` reads the task worktree's status through `state.ReadJSONStrict` over `shedrun.StatusFile` and `shedrun.StatusLock`.
`ReadJSONStrict` takes the read lock first (`internal/state/state.go:147`), and `lock.AcquireReadLock` opens the lock file through gofrs/flock with `os.O_CREATE`.
That open fails with ENOENT when the task worktree is gone, and the composite already supports that route (`GonePairSkipsQuietWaitAndEndsByName` in `TestTeardown_CallSequence`).
Under the card's rule, "a board or status read failure warns and writes nothing", so a pair whose task worktree is already gone keeps its run-form claim whatever batten's state is.
D10 requires that "every teardown route releases the claim by construction".
The card's phrase "while the task worktree is present" does not say what `landed` is when the worktree is absent.

**Fix:** State in the Intent that `childLanded` reports not-landed with no error when the task worktree is gone (checked through the composite's existing `gone` substrate), so the settle falls through to the batten-state rule.
Add a `TestTeardown_CallSequence` row for a gone pair over a run-form entry with no batten seed, asserting the status is cleared.

### [MEDIUM] The real claim substrates are reached by no test

**Location:** `_lyx/plan/03-teardown-settles-board-claim.md:45`

**Issue:** `TestTeardown_CallSequence` replaces all four new substrates (`landed`, `battenRun`, `readClaim`, `writeClaim`) with fakes.
The batten lifecycle step reaches `readBoardClaim` and `writeBoardClaim` for real, but only through the request's landed flag, so `settleBoardClaim` never calls `battenRunState` there.
That step's child status also holds no `Finalize` Done, so `childLanded` never returns true against a real file.
No planned test runs `childLanded` against a real status file holding a `Finalize` Done, or `battenRunState`'s `shedrun.ReadSeed` and status read on the prime.
These are the file reads that decide the acceptance criteria for `lyx fabric remove`: clear on no seed or halted batten, done on batten `done` or a `Finalize` Done, held on `running`.
A wrong decode type or path in either reader would ship green.

**Fix:** Add `internal/pairteardown/teardown_integration_test.go` to card 3's `Edit:` list.
Give `TestRun_TeardownScenario` steps over a hubforge pair with a run-form board entry that cover three cases.
With no batten seed, the entry is cleared after `Run`.
With a child status whose history holds a `Finalize` Done, the entry is marked done.
With a prime batten status of `running`, the entry is left unchanged.
That file is `tmux`-tier, which the plan compiles and Publish runs, so no `## verify:` change is needed.

### [MEDIUM] Card 13's darn-seed bound disagrees with card 12's

**Location:** `_lyx/plan/13-batten-admits-darn.md:20`

**Issue:** Card 13 says the admit set is "the bound on which runs carry a darn seed: only a board entry whose operator-set `recipe` is `darn` seeds one".
Card 12's bound (`12-loom-arms-darn.md:40`) also names the hand route `lyx shed seed`.
Card 7 makes that route admit `darn` with no seed-location rule, the same as loom's row in `internal/shedcli/table.go`.
So card 13's "only" is false once card 7 lands, and the two cards that open the route state different bounds.

**Fix:** Make card 13's bound sentence match card 12's: batten's child seeding admits `darn` only from the board entry's operator-set `recipe`, and the one other writer of a darn seed is a hand `lyx shed seed`.

### [LOW] The merge-base read error has no stated stop in the whole-diff form

**Location:** `_lyx/plan/06-verify-gate-told-command.md:40`

**Issue:** The whole-diff form states its stops for a command read error and an empty command, but not for an error from the told `mergeBase` reader.
That reader is `fabricengine.Open` then `Fabric.MergeBase` per card 12, and card 5 makes "no common ancestor" an error.
The round form's lint is conditional on a non-empty base (`derivation.Base != ""` in `internal/loomshed/verifygate.go`), so an implementation that mirrors it could treat an empty or failed base as "skip the lint".
That would let a darn run through with fixed-column comment wraps the gate is meant to catch.
None of the planned gate scenarios reaches this case.

**Fix:** State that a `mergeBase` read error is returned as the gate's error with its message kept, never a lint skip.
Add a step to the loomshed verify gate scenarios with a failing `mergeBase` reader, asserting the gate returns that error and runs nothing.

## Verdicts on the focus directives

1. Refusal-spec rows: every D13 row has a card.
   Card 12 adds `darn.yaml: verify missing or empty`, edits the `Publish: verify failed` way forward, and edits the `gate slot: gate.yaml absent or invalid` trigger to name the `Darn` row's verify gate.
   Card 11 adds `verify_attempts below 1`, card 10 adds `Darn verify gate exhausted`, and card 2 adds the Finalize commit row.
   Not a defect.
2. Tests: card 3 drives the rule through the exported `Run` and extends `TestTeardown_CallSequence` rather than adding a test, though see F1 and F2.
   No `internal/battencli/teardown_test.go` is planned, and card 13 extends `TestWire_WriteSeedRefusesANonLoomChildBeforeTouchingTheWorktree` in `wire_test.go`.
   Card 11 makes `TestRecipe_StructuralCheckHasNoFindings` a table over both recipes and unions `TestInterruptPolicies_MatchAssembledRows` over `New` and `NewDarn`.
   Its new `darnrecipe_test.go` pins `NewDarn`, a separate exported builder whose shape, stamped verify budget and refusal no existing test covers.
   Not a defect.
3. Whole-diff gate: the Intent bounds the empty and unreadable command (an error, no skip path), and the tests reach the read-error stop, the empty-command stop and the Publish failure-record branch.
   Not a defect; F4 covers the remaining merge-base stop.
4. Board-claim clear: card 3's Intent states what the clear lets through and what bounds it, matching D10.
   Not a defect.
5. Darn seed bound: card 12 states the skipped reviews and the bound; card 13's bound disagrees with it (F3).
6. Verify log path: card 1 lists every non-test reader I found of `Paths.Log` and `Paths.PublishFailureLog`.
   Those are `loomshed.NewVerifyGate` and `checkedPublishFailure`, landing's `recordFailure` and `verdict`, websterengine's gate seams and webstercli's `runVerify`.
   It also lists every test file that reads them: the loomshed, landingshed, verifytree, websterengine and webstercli tests.
   `internal/loomshed/verifygate_test.go` and `internal/websterengine/verifygatereport_test.go` use literal temp paths that are independent of `Paths`.
   Not a defect.
