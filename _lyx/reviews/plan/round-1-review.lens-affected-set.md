```yaml
lens: affected-set
outcome: reported
```

Fork task: affected-set lens. For each card's change, list what reads or asserts it and which cards cover it.

## Affected sets per change

- **Card 13: Run-Shed becomes one check per Call that ends in `Yield`, with the memory in a record written through `internal/state`.**
  - Found: the battenshed InnerRun and notice tests (card 13 covers them).
  - Found: `docs/overview.md`, `battenshed/doc.go` and the recipe header (card 13 covers them).
  - Found: `cmd/lyx#unboundedWaitFiles` and `pattern/PATTERN-daemon-wakeups.md` (card 32 covers them).
  - Found: battencli tier1 tests that assume the in-call wait. These are `stepStepDrivenRunShed_ReturnsOnTheChildsStateChange`, `stepRunShedPausedChild_WaitsThenTearsDownOnceDone`, the `wireForHub` fixture and `stepAwaitingChildDoneTeardown_ArchivesTheRunRecords`. **No card covers them.**
  - Found: the battenshed import allowlist `battenshedAllowedImports`. **No card covers it.**
  - Found: `pattern/PATTERN-shed-driver.md` "Outside it", which still describes "Batten's InnerRun wait". **No card covers it.**
- **Card 28: the `--window` and `--open-terminal` flags, the `windowFlag` and `openTerminalFlag` fields and `ResumeInWindow` are retired.**
  - Found: `runInWindow`, the window tests, `wire`, `TestBattenRun_RefusesWhileAPauseCoversTheSlug`, `TestArmAt_StartGuardRefusesBeforeAnyWrite`, `adoptcli` `hubDepsAs`/`adoptDeps`, `beamcli` `hubLandDeps`/`landDeps`/`landCmd`, and `docs/overview.md` lines 628 and 639 (card 28 covers them).
  - Found: `internal/battencli#battenCLI.registerSeededRun`, which reads `c.windowFlag`. **Missed.**
  - Found: `TestWire_MarkerAndReportSeamsRefuseAnAbsentTaskWorktree`, `TestArmSeed_RunAndStepSeedBeforeWire` and `stepCommandRows_FollowTheTasksLocalSwitch`. **Missed.**
  - Found: `internal/adoptcli/doc.go:45` and `internal/beamcli/doc.go:59`. Cards 39 and 37 fix these later, so they are stale from card 28 to those cards.
  - Found: the orch stencil's `--window` lines (card 42 covers them).
- **Card 17: the ready seams, the ready helpers and the `landed` notice go.**
  - Found: `LandDeps`, the `land_test` rows, `battencli` `land.go`/`land_test.go` (`TestReadySlugs`, `TestReadyRoutes`), `envkit`, `shedrecipe.landEntry`, the docs, and orch stencil lines 140 and 144.
  - Covered by cards 17 and 42. `creelengine/deliver_test.go` uses the text only as sample data.
- **Card 5: new tasks become `draft`.**
  - Found: the Board-level tests that assert a status are `TestSetRunStatus`, `TestWire_CreateWorktreeClaimsTheBoardEntry` and `fourRowRunForRecipe`. They all assert non-nil run or done values, so they are unaffected.
  - Found: boardcli `set_status_null_status_clears`. It still passes, since a release reads nil.
  - Found: the fixtures that start fresh runs (card 7 covers them).
  - The cover is complete.
- **Card 3: `ender.end` takes a variadic `spare`.**
  - Found: `End` and `end_test.go:407`, inside `TestEnd` (card 3 covers them). The variadic keeps both compiling.
- **Card 40: the `endOwned` seam is re-typed.**
  - Found: `New` (teardown.go:164) and `fakeSubstrates.teardown` (teardown_test.go:93), both covered by card 40.
  - Found: `cancel.go:149` `t.endOwned(slug)`. The variadic keeps it compiling.
  - The cover is complete.
- **Card 4: `dobbyengine` joins the strict config loaders.**
  - Found: `cmd/lyx#configStrictnessStrictSet` (card 4 covers it).
  - Found: the strict-callers list in `pattern/PATTERN-config-layers.md`. No card covers it, but that list is already out of date: it lacks `darnengine`.

## Findings

1. **BLOCKING · design · card 13 (`13-innerrun-short-check.md`)**
   The card turns the Run-Shed wait into one check that yields, but leaves out the battencli tier1 tests that pin the in-call wait.
   - Missed: `internal/battencli#stepStepDrivenRunShed_ReturnsOnTheChildsStateChange`. It asserts a blocking step of at least 1.8 s that ends in Stuck (`internal/battencli/lifecycle_integration_test.go:782-856`).
   - Missed: `internal/battencli#stepRunShedPausedChild_WaitsThenTearsDownOnceDone`. It moves the child between in-call checks through `InnerRun.Sleep` (`lifecycle_integration_test.go:861-925`).
   - Missed: `internal/battencli#wireForHub`. It relies on the pause flag after the first sleep to end the wait (`lifecycle_integration_test.go:116-128`).
   - Missed: `internal/battencli#stepAwaitingChildDoneTeardown_ArchivesTheRunRecords`. It asserts `Outcome == Stuck` for an awaiting child (`internal/battencli/records_integration_test.go:105`).
   - With one check per Call, each of these gets `Yield` instead. The fix adds these members to card 13's `**Edit:**`.
2. **BLOCKING · consistency · card 13**
   - `innerRunMemory` is written through `internal/state`, but battenshed's production code does not import `internal/state` today.
   - The closed allowlist that `scankit.AssertImportAllowlistNoStale` checks lacks it (`internal/battenshed/seam_enforcement_test.go:23-31`).
   - The card does not list `internal/battenshed#battenshedAllowedImports`, so `TestToldGeometryInvariant_AllowlistOnly` fails on card 13.
3. **BLOCKING · design · card 28 (`28-batten-first-pass.md`)**
   Removing the window flags from `battenCLI` misses members that read them:
   - `internal/battencli#battenCLI.registerSeededRun`, which reads `c.windowFlag`. This is production code, so the build breaks (`internal/battencli/arm.go:296`).
   - `internal/battencli#TestWire_MarkerAndReportSeamsRefuseAnAbsentTaskWorktree`, which sets `openTerminalFlag: true` (`wire_seams_test.go:256`).
   - `internal/battencli#TestArmSeed_RunAndStepSeedBeforeWire`, which sets `windowFlag: true` (`arm_seed_test.go:70`).
   - `internal/battencli#stepCommandRows_FollowTheTasksLocalSwitch`, which sets `openTerminalFlag: true` (`lifecycle_integration_test.go:1314`).
4. **LOW · consistency · card 28**
   - `internal/adoptcli/doc.go:45` and `internal/beamcli/doc.go:59` still name `battencli.ResumeInWindow` after card 28 deletes it.
   - Cards 39 and 37 rewrite those docs later, so the module docs stay stale across cards 28 to 39, against the docs-in-the-same-commit rule.
5. **LOW · consistency · card 13**
   - `pattern/PATTERN-shed-driver.md` "Outside it" says "Batten's InnerRun wait polls a child run it does not step". After card 13 there is no wait, only one check per pass.
   - No card updates the background file.

## Tool calls

1. Grepped `--window`/`--open-terminal` in docs, plugins, contracts and pattern, and grepped the ready-notice readers, to find text readers the pre-read lacked.
2. Grepped battencli `land_test.go` functions, Board-level tests asserting a status, and README goldens, to complete card 17's and card 5's sets.
3. Named the functions enclosing the status asserts in those files, to tell whether any asserts nil after a create.
4. Grepped tests asserting a JSON `"status": null`, plus production nil-status readers in other packages, for card 5's output change.
5. Read the boardcli `set_status_null_status_clears` row, to judge whether a release still passes it.
6. Grepped `ender.end` callers and `endOwned` seam fills, to complete card 3's and card 40's re-signature sets.
