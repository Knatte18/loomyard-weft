---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lyx/plan/02-shared-hub-board.md
    summary: TestSetRunStatus's locked-entry row re-pins the status-only pass that Card 1's TestRunLock SetStatus rows already cover
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/03-batten-claim.md
    summary: Card 3's no-board-config row is said to return the same fabric-config error as the first row, but it gets a different error, and the row never asserts the warning that is its point
  - id: F3
    severity: NIT
    class: consistency
    location: _lyx/plan/02-shared-hub-board.md
    summary: markDoneFixtureDir moves to the board kit but keeps its own copy of OpenHub's body to build the board handle
---

### [LOW] TestSetRunStatus's locked-entry row repeats a behavior Card 1 already pins

**Location:** `_lyx/plan/02-shared-hub-board.md`, Test paragraph, row "a locked entry (status replaced, which the Card 1 lock lets through)".

**Issue:** Card 1's `checkRunLocks` judges every write by its net change in `boardCriticalSection`, whichever `Board` method made it.
Card 1's `TestRunLock` already has passing rows for `SetStatus` to another value and to nil on each seeded run status.
`SetRunStatus` writes `status` alone, so its locked-entry row checks the same status-only pass again.
The only new behavior in Card 2 is the absent/`done` skip.
None of the existing loom tests covers that skip: `wiring_commitstatus_test.go` stubs `SetBoardStatus`, and `wiring_commitstatus_integration_test.go` makes no board assertions.
`PATTERN-test-economy` says a row must cover something no other test covers.

**Fix:** Drop the locked-entry row from `TestSetRunStatus` and keep the no-status, `done` and absent rows.

### [LOW] Card 3's second claim row describes the wrong error and leaves the warning unasserted

**Location:** `_lyx/plan/03-batten-claim.md`, Test paragraph, row "a bare `t.TempDir()` hub with no board config returns the same fabric-config error and writes nothing".

**Issue:** In the first row, the hub from `HubWithBoardConfig` has `_board/_lyx/` but no `fabric.yaml`.
`configengine.load` therefore returns `config file … not found; run "lyx config reconcile"` (`internal/configengine/config.go:140-145`).
In the bare `t.TempDir()` hub there is no `_board/_lyx/`, so `FindBaseDir` fails first with `ErrNotInitialized` (`config.go:61-65`).
The two rows do not get the same error, and an implementer who follows "the same" literally writes an assertion that fails.
The row exists to show that a failed claim write warns and the run continues (Decision 8; acceptance criterion "a failing board write warns and the run proceeds").
"Writes nothing" checks nothing, because the hub has no board to write.
The row asserts only that the closure got past the claim, never that the warning was logged, even though a log-capture kit exists (`internal/testkit/logcapture`).

**Fix:** Have the row assert that the closure's error is non-nil and is not the board's error, which shows the run went past the claim.
Have it also assert, through `logcapture`, a `logger.Warn` that names the slug.
Remove "the same fabric-config error" and "writes nothing".

### [NIT] markDoneFixtureDir keeps a copy of OpenHub's body

**Location:** `_lyx/plan/02-shared-hub-board.md`, the `HubWithBoardConfig` bullet ("`markDoneFixtureDir` … builds its hub through it").

**Issue:** `markDoneFixtureDir` currently builds its board handle with `LoadConfig(boardDir, "board")`, sets `Path`, then calls `New(ApplySkipEnv(bc))` (`internal/loomcli/landingdeps_markdone_test.go:41-46`).
That is the body Card 2 moves into `boardengine.OpenHub`.
The card moves only the config writing into the kit, so the test would still hold a copy of the opener it now shares with production.

**Fix:** Also say that `markDoneFixtureDir` gets its board handle from `boardengine.OpenHub(location.HubPath)`.

## Focus verdicts

- Card 3's board-config fixture against `PATTERN-testkit`: no defect.
  Today only `internal/loomcli/landingdeps_markdone_test.go:35` writes `boardengine.ConfigTemplate()` into a board directory.
  Card 2 moves that write into `internal/testkit/boardkit#HubWithBoardConfig` and has both loomcli and battencli use it, which matches the kit rule for a fixture shared by two packages.
  The kit imports `boardengine`, `configengine` and `fabricengine`, and none is on the `enforcement_test.go` deny list.
  `fabricengine` does not import `boardengine`, so there is no cycle.
  Other tests write a hand-written board config with their own content rather than the template: `boardcli`'s `seedHubBoardConfig`, `ideengine`'s `writeBoard`, `cmd/lyx/main_integration_test.go` and `boardtest/bench_test.go`.
  Those are not copies of this helper.
- Card 3 and a `Topology.Add` refusal: Card 3 reaches it.
  It extends `stepCreateRow_LeftoverBranchIsRewordedForPrime` (`internal/battencli/lifecycle_integration_test.go:873`), which already sends `c.env.CreateWorktree` into `top.Add`'s leftover-branch refusal through `createRefusal`.
  The step seeds the entry with `seedBoardTask` and checks that it reads `running · Worktree-Create` after the refusal.
  It adds an assertion and no new top-level test.
  This is not redundant with the untagged claim test: that test fails at the fabric config load, before `Add`, so it never shows that an `Add` refusal leaves the claim in place.
  The acceptance criterion names that case.
- Re-pinned rows in Cards 1, 2 and 3: one found, F1.
  Card 1's rows match the acceptance criteria one for one.
  Its unlocked-entry behavior goes through `IsRunStatus` rows plus a single `active` row, not a row per write.
  Card 3 leaves the absent/`done` skips to Card 2.
  Test-budget raises fit the current counts: `boardengine` 73/73 rises by two over Cards 1 and 2, and `battencli` 57/57 rises by one.
