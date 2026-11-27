# Run webstercli and loomcli tier-1 tests in parallel over shared pairs

`internal/webstercli` and `internal/loomcli` were two of the slowest packages at tier 1.
In `webstercli` most tests ran one after another because they set process environment variables, which bars `t.Parallel`.
In `loomcli` nearly every hub test built its own task pair, and a set of tests ran serially for no stated reason.
No assertion is dropped.

## webstercli: values passed in instead of read from the environment

- `websterCLI` gains two fields that wiring fills once per process: `syncOptions` (from `fabricengine.EnvSyncOptions`) and `strandName` (from the strand-name variable).
- `fabricSync` takes the `fabricengine.SyncOptions` as an argument instead of reading the environment, and every verb passes `c.syncOptions`.
  `reset` passes the same options to `ResetPairCode`.
- `envMasterStrand` becomes `masterStrandLookup(name, resolve)`.
  The orphaned-Master guard of the bracket verbs reads `c.strandName`.
  Production behaviour is unchanged: the same variables are read, at wiring instead of at call time, within one process.
- Tests set the fields on the `websterCLI` they build, and call `t.Parallel`.
- The verify tests put the built `lym` first on `PATH`.
  The six `TestVerifyVerb_*` tests are now subtests of one `TestVerifyVerb`, which sets `PATH` once and runs them in parallel.
  `TestGateParity_VerifyGate` does the same for its cases.
- `TestResetCmd`'s cases added their pairs to one shared hub in sequence, and that made the test the package's critical path.
  Each case now copies the hub template that already holds a pair, and the cases run in parallel.
- Still serial: the tests that set `XDG_STATE_HOME`, `LOCALAPPDATA` or `LYM_TRACE`, or call `t.Chdir`, because production reads those through code this change leaves alone.
  `TestBatchVerbs_RefuseUnderAPause` also stays serial, since its rows capture the process-wide logger.

## loomcli: one built pair shared through the template, more tests in parallel

- Tests that copied a bare hub and then added one pair now copy `hubforge.Shape{Anchor: ".", Pairs: []string{hubforge.TemplatePairSlug}}`.
  That template's pair is built once per test binary by the same `AddPair`, so a test copies it instead of building one.
  Their slugs become `hubforge.TemplatePairSlug`.
  A test that writes into the hub before its `AddPair`, or that uses `AddPairWith`, keeps its own pair.
- Rows that each added a pair to one shared copy (`TestApproveDiscussion_Real`, `TestSeedDiscussedRecord_Real` and both tests in `commitstatus_integration_test.go`) take a copy each.
  They now run as parallel subtests under their serial parent, which still sets `BOARD_SKIP_GIT`.
- Serial tests that touch no process-global state now call `t.Parallel`, for example `TestCirclingCLI_PreRunResolvesTarget`, `TestCommitStatusSeam_Real` and `TestWriteCarryOver_Real_CommitsTheRecordAlone`.
- `TestWireLanding_TellsLanguageSkills` and `TestWireLanding_FillsSamplerSessionSeams` had the same shape.
  They are now one table, `TestWireLanding_FillsTheLandingSeamsPerRecipe`, over one hub copy, and each row checks both tests' assertions.
- `halt_test.go`, `pusher_test.go` and `pusher_integration_test.go` are left as they were.

## Measurements

The package's time from go's `ok` line, for `lym gate test --wait --no-carry --tier 1 ./internal/<pkg> -- -count=1 -v`.
The calls ran without `--gate`, because a `--gate` call takes no `--tier`, the same way as in `docs/research/speed/test-time-measure-2026-11-27/report.md`.
Each run is listed with its 1-minute load average at the start, on a 12-thread machine.

| Package | Before, run 1 | Before, run 2 | After, run 1 | After, run 2 |
|---|---|---|---|---|
| `internal/webstercli` | 13.1 s (load 16.1) | 12.2 s (load 25.5) | 6.8 s (load 13.9) | 5.7 s (load 15.1) |
| `internal/loomcli` | 17.8 s (load 16.5) | 7.4 s (load 19.9) | 5.4 s (load 15.6) | 8.2 s (load 18.2) |

`loomcli`'s time swings with load from run to run: the two before runs differ by more than 2×.
Its after runs stay at or below the better before run.
In `webstercli` the serial tests summed to about 11 s before.
After the change only `TestCommand_WronglyEnteredHub_Refuses` (0.7 s) and a few short tests run serially.

## Merge danger

**Door:** two-way
**Blast radius:** webster
