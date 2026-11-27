# Run more webstercli and loomcli tests in parallel over copied pairs

`docs/research/speed/test-time-measure-2026-11-27/report.md` measured `internal/webstercli` and `internal/loomcli` as slow packages.
In `webstercli`, tests that set process environment variables could not call `t.Parallel`.
In `loomcli`, most hub tests built their own task pair, and several tests ran serially without touching process state.
No assertion is removed, though one `loomcli` wiring case no longer runs, as described below.

## webstercli: values passed in instead of read from the environment

- `websterCLI` gains `syncOptions` (from `fabricengine.EnvSyncOptions`) and `strandName` (from the strand-name variable),
  both filled by `wireHub` and `wireStandalone`.
- `fabricSync` takes the `fabricengine.SyncOptions` as an argument, and `reset` passes the same options to `ResetPairCode`.
- `envMasterStrand` becomes `masterStrandLookup(name, resolve)`, and the orphaned-Master guard reads `c.strandName`.
  The same variables are read as before, once at wiring instead of at each use.
- Tests set the fields on the `websterCLI` they build and call `t.Parallel`.
  `TestWire_ModeHubSelectsHubMode` and `TestWire_StandaloneMode` now also assert that wiring reads the variables.
- The `TestVerifyVerb_*` tests are now subtests of one `TestVerifyVerb`, which puts the built `lym` first on `PATH` once
  and runs them in parallel. `TestGateParity_VerifyGate` does the same for its cases.
- `TestResetCmd`, the package's longest test, ran its cases in sequence over one shared hub.
  Each case now copies the hub template that already holds a pair, and the cases run in parallel.
- Not every test is parallel now. Those still serial include:
  - tests that set `XDG_STATE_HOME`, `LOCALAPPDATA` or `LYM_TRACE`, or call `t.Chdir`,
    because production reads those through code this change leaves alone;
  - the cases in `wiring_test.go`, including those that set none of these,
    such as the two that now set the skip switches and the strand-name variable;
  - `TestWire_HubLoadsSecretPathsFromFabricYAML`, because wiring sets the process-wide logger sink;
  - `TestBatchVerbs_RefuseUnderAPause`, because its rows capture the process-wide logger.

## loomcli: pairs copied from the template, more tests in parallel

- Tests that copied a bare hub and added one pair now copy a hub shape holding `hubforge.TemplatePairSlug`.
  The pair is built once per test binary.
  Some tests still add their own pair, for example those that seed config into the hub before `AddPair`.
- Each test still takes its own copy through `hubforge.CopyHub` rather than `hubforge.SharedHub`,
  because the moved tests write into the hub or spawn git in it, which `SharedHub` treats as a write.
- Rows that each added a pair to one shared hub (`TestApproveDiscussion_Real`, `TestSeedDiscussedRecord_Real`
  and the tests in `commitstatus_integration_test.go`) now take a copy each and run as parallel subtests.
  Their parents stay serial because they set `BOARD_SKIP_GIT`.
- Several serial tests that touch no process-global state now call `t.Parallel`; others in the package remain serial.
- `TestWireLanding_TellsLanguageSkills` and `TestWireLanding_FillsSamplerSessionSeams` are now one table,
  `TestWireLanding_FillsTheLandingSeamsPerRecipe`, over one hub copy. Each row checks both tests' assertions.
  The rows are the sampler and darn recipes, so the language-skill and PR-Review seam assertions
  no longer run against a `loomCLI` with no `recipe`, as `TestWireLanding_TellsLanguageSkills` did.

## Test time

The time on go's `ok` line for an uncached run of each package, with tests up to `tiergate` level 1,
each with the 1-minute load average at its start on a 12-thread machine.
The "after" runs predate the last edits: formatting, comments, test assertions, a test-only export,
and a fixture change that moves `TestRunOwnerVerdict` onto the shared pair shape, which can affect the `loomcli` time.

| Package | Before, run 1 | Before, run 2 | After, run 1 | After, run 2 |
|---|---|---|---|---|
| `internal/webstercli` | 13.1 s (load 16.1) | 12.2 s (load 25.5) | 6.8 s (load 13.9) | 5.7 s (load 15.1) |
| `internal/loomcli` | 17.8 s (load 16.5) | 7.4 s (load 19.9) | 5.4 s (load 15.6) | 8.2 s (load 18.2) |

Both `webstercli` runs after the change are faster than both runs before it.
The `loomcli` figures show no clear gain: the two runs before differ by more than 2×,
and the second run after (8.2 s) is slower than the second run before (7.4 s).

## To check by hand

- The `loomcli` timing statement against the table above, in "Test time":
  these figures do not establish that `loomcli` got faster.
- Whether it is acceptable that `internal/webstercli` still reads `XDG_STATE_HOME`, `LOCALAPPDATA`, `LYM_TRACE`
  and the working directory from the process, so the tests that change them stay serial
  (in `wiring_test.go`, `cli_test.go`, `cli_integration_test.go` and `cli_tmux_test.go`).
- The environment read in `internal/webstercli/wiring.go`:
  confirm that the assertions in `TestWire_ModeHubSelectsHubMode` and `TestWire_StandaloneMode`
  cover `syncOptions` and `strandName` in both wiring paths, since every other test sets the fields directly.

## Merge danger

**Door:** two-way
**Blast radius:** webstercli
