# lyx temp root and `lyx cleanup`

All five batches (cards 1–20) recorded `done`, final HEAD `1c0be248c`.

## Batches

- **01-lyxtemp-package** (head `07949ed22`): added `internal/lyxtemp` (per-user root, locked run directories, sweep, process API).
  `tmuxkit.Main` runs in the process run directory, gitkit writes its hermetic gitconfig and repo template there, hubforge, lyxbin, ide and reed test temp directories moved in, and the gate's prebuilt `lyx` is built there and removed in `lyx`'s main.
- **06-testtiming-run-directory** (head `e23de91e1`): testtiming's directories moved into the run directory, leftovers are attributed by the inherited `LYX_TESTTIMING_PACKAGE` marker instead of a per-package `TMPDIR`, and 26 internal test packages a to l got a `TestMain` calling `tmuxkit.Main`.
- **08-testmain-internal-m-z** (head `7332207175`): 33 internal test packages m to z got the same `TestMain`.
- **09-testmain-kits-tools-contracts** (head `60f541401`): `TestMain` added to kit, tools and contracts packages, and gitoracle's moved under the kit call.
  The isolation guard now requires `tmuxkit.Main` in every test package and was renamed `TestTmuxIsolation_EveryTestPackageRunsThroughTmuxkitMain`.
  A bare-temp scan (`cmd/lyx/baretemp_test.go`) and `PATTERN-lyx-temp-root` were added.
  Logger's sweep gained a dry run and `loggerconfig` the exit sweep's bound choice, and shedrun gained the run-records roots and the run directory's relative path.
- **15-fsx-newest-modtime** (head `1c0be248c`): added `fsx.NewestModTime`, `lyx batten cleanup`, the done-slug refusal naming `lyx batten cleanup --apply`, `lyx orch cleanup`, the `internal/cleanupcli` package with its `.scratch` segment, and `lyx cleanup` mounted in the root.

## Deviations from declared file-ops

Batch 01 changed files outside its cards' targets: `internal/gitkit/leaf_enforcement_test.go`, `internal/gatecli/test_integration_test.go`, `pattern/PATTERN-leaf-packages.md`, `pattern/PATTERN-test-isolation.md`, `docs/overview.md` and `cmd/lyx/testdata/test-budget.yaml`.
The other batches reported none.
Every batch's `warnings` were informational scope notes only.

## Notes from the fork reports

- Batch 09's card gates listed `./internal/logger_test` and `./internal/loggerconfig_test`, which are not directories, so `lyx gate test` reported `[setup failed]` for them. The real packages passed. The plan's glyph qualifiers need correcting.
- Batch 15 raised the `internal/battencli` test-budget row from 65 to 66, since card 16's own gate never ran `./cmd/lyx`.
- Two forks edited files with a shell heredoc rather than Edit or Write (batch 06's new test-main files, batch 15's append to `internal/orchcli/cli_test.go`).


## Integration suite triage

The plan-level `## verify:` suite failed, but webster's triage did not attribute the failure to this run.

Flaky (passed on rerun):

- `github.com/Knatte18/loomyard/internal/lyxtemp.TestSweep/two_concurrent_sweeps_both_succeed`
