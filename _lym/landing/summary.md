# Move test-only functions out of production code and guard against them

`deadcode` listed functions in production files that no `main` package of the module reaches.
Most were test seams or helpers that only tests called.
This change removes them, moves the ones tests still need into `_test.go` files,
and adds `PATTERN-test-only-code` with a guard test that fails on a new one.

## What changed

- **`RunCLI` and `RunCLIIn` are gone from every `*cli` package.**
  Each was a thin wrapper over `clihelp.Execute(Command(), ...)` or `clihelp.ExecuteIn(Command(), ...)`.
  Every caller now names `clihelp.Execute(<pkg>.Command(), out, args)` or `clihelp.ExecuteIn(<pkg>.Command(), cwd, out, args)`,
  so the tests run the same code as before.
  A `reedcli` smoke helper whose cwd can be empty picks `Execute` for that case,
  as `RunCLIIn("")` did.
  `configcli`'s sync, the one production caller of `fabriccli.RunCLIIn`, calls `ExecuteIn` the same way.
  `cmd/lym/seamsignature_test.go`, which pinned the wrappers' signatures, is deleted,
  and `PATTERN-cli-cobra` and `docs/overview.md` say how a test runs a module in process.
- **`cmd/lym`'s `run` and `main` share one function.**
  `run(args, stdout, stderr)` builds the root and runs it, and `main` calls it with the process streams.
  Tests pass one buffer as both writers.
- **The other unreachable functions are deleted or moved.**
  A function nothing live needed is deleted with the tests that tested only it,
  such as `logger`'s span API, `orchengine.CleanupRetiredNotices`, `fabricengine.PushWarpAt` and `shedbuild.Load`.
  A test of another package that called a deleted function now calls the live function with the same meaning:
  `PostWith` for `creelengine`'s `Post`, `WireJunctionsWith(nil, ...)` for `WireJunctions`,
  `PushPairAnchored` on the prime for `PushAnchored`.
  `reedengine.tmuxProcessName`, which only the Windows process-tree code calls,
  moves into `proctree_windows.go`, and its test into a Windows-only test file.
- **`PATTERN-test-only-code`** says a function only tests call lives in a `_test.go` file
  or a test-support package (`internal/testkit`, `internal/gitkit`, `internal/hubforge`).
  `cmd/lym/testonlycode_linux_test.go` (tier 1, `//lym:guard`) enforces it.
  It runs `deadcode`'s analysis in process through `golang.org/x/tools`:
  rapid type analysis over SSA, rooted at every `main` package under `cmd/` and `tools/`,
  and at every function of a test-support package, so a function only `hubforge` calls counts as reached.

## Functions kept in `_test.go` files

These moved instead of being deleted, because tests of live code use them.

References or oracles that tests compare live behaviour against:

- `pattern.Check` and its helpers, the checker run over the live `PATTERN.md`.
- `planglyph.ValidateFormat`, `ValidateRework` and `Validate`, thin wrappers over the live validators.
- `preflight.Wired`, for the predicate-split tests.
- `shuttleengine.IsDraftHold`, the error classifier in the send tests.
- `fabricengine`'s `snapshotWarpSHA`, as `SnapshotWarpSHAForTest`.
- `loomshed.ArchivedPlanWebsterDirs`, for the plan rotation test.
- `tokencount`'s `RunTally.RoleTime`, which the accounting tests compare against.

Setup helpers:

- `boardengine`'s `Board.Promote` and `Store.Promote`.
- `creelengine`'s `Post`, `PostWait` and `PostInterrupt`.
- `agentconfig.Roles`, `discussionparser.AppendDecision` and `gateslot.TemplateConfig`.
- `fabricengine`'s `commitWeft`, `WireJunctions`, `CodeLymLink` and `PushAnchored`.
- `websterengine`'s template accessors and `codexengine.New`.
- `preflightshed.NewPreflight` and `godocreflow`'s `reflowSource`.

## Design decisions

- **`gitkit` and `hubforge` count as test-support packages beside `internal/testkit`.**
  The task asked for `_test.go` files or `internal/testkit`.
  `gitkit` spawns git and `hubforge` builds hubs through `fabriccli`,
  and `PATTERN-test-isolation` bars both imports from a kit, so neither can move there.
- **References other packages' tests share stay in production, each named in the guard's allowlist.**
  A `_test.go` file of the defining package cannot serve another package's tests.
  Each of these either reads its package's own state, such as `claudeengine.New`,
  or writes a record its package alone writes, such as `driftengine.UpdateScope` and `websterengine.WriteReport`.
  Every entry of the allowlist names the tests it serves, and an entry that matches nothing fails the guard.
  They are:
  - `shedcheck.Check` with its helpers, and `shedbuild.Check`.
  - `shedrecipe.Names` and both `RecipeEngines`.
  - The four `fabricvocab` checks and `fabricengine.RecordPushFailure`.
  - `seatengine.Prompts` and `shortcutcli.Load`.
  - `slackengine.WriteSetup` and `claudeengine.New`.
  - `driftengine.UpdateScope`.
  - `websterengine`'s `ConflictRecordRel`, `Fingerprint` and `WriteReport`.
  Moving the pure ones, such as the recipe checkers, into a kit is possible follow-up work.
- **The guard compiles on Linux only.**
  The analysis type-checks one build, and this machine cannot type-check the Windows build of `quarry`'s cgo code.
  A function only a Windows file calls must sit in a Windows-only file, as `tmuxProcessName` now does.
- **Safety checks before deleting.**
  `deadcode` ran rooted at `./cmd/...` and `./tools/...` under `GOOS=linux`.
  It could not run under `GOOS=windows`, where `quarry` needs a cgo cross toolchain,
  or under `GOOS=darwin`, where `internal/proc` has no implementation.
  In their place, every candidate's name was checked against every file only Windows or darwin compiles;
  that found `tmuxProcessName` alone.
  Each candidate's name was also checked for reflection, `go:linkname`, template function maps and string lookups,
  and none was reached that way.
  The touched production packages build under `GOOS=windows`.
