# Move test-only functions out of production code and guard against them

`deadcode` listed functions in production files that no `main` package of the module reaches.
Most were test seams or helpers that only tests called.
This change removes them, moves the ones tests still need into `_test.go` files or a kit,
and adds `PATTERN-test-only-code` with a guard test that fails on a new one.

Closes #123

## What changed

- **`RunCLI` and `RunCLIIn` are gone from every `*cli` package.**
  Each was a thin wrapper over `clihelp.Execute(Command(), ...)` or `clihelp.ExecuteIn(Command(), ...)`.
  Every caller now names `clihelp.Execute(<pkg>.Command(), out, args)`
  or `clihelp.ExecuteIn(<pkg>.Command(), cwd, out, args)`, so the tests run the same code as before.
  `configcli`'s sync, the one production caller of `fabriccli.RunCLIIn`, calls `ExecuteIn` the same way.
  `cmd/lym/seamsignature_test.go`, which pinned the wrappers' signatures, is deleted,
  and `PATTERN-cli-cobra` and `docs/overview.md` say how a test runs a module in process.
- **`cmd/lym`'s `run` and `main` share one function.**
  `run(args, stdout, stderr)` builds the root and runs it, and `main` calls it with the process streams.
  Tests pass one buffer as both writers.
- **The other unreachable functions are deleted or moved.**
  A function nothing live needed is deleted with the tests that tested only it,
  such as `logger`'s span API, `fslink`'s unused helpers, `shedbuild.Load` and `shedbuild.Check`.
  A test of another package that called a deleted function now calls the live function with the same meaning.
  `reedengine.tmuxProcessName`, which only the Windows process-tree code calls,
  moves into `proctree_windows.go`, and its test into a Windows-only test file.
- **`internal/shedcheck` is now `internal/testkit/shedcheckkit`.**
  It is a pure checker only tests run, so it moved into a kit instead of staying in production under an allowlist entry.
  `battenrecipe.RecipeEngines` moved into the kit too, and `fabricvocab.FailsBareCheck` into `lymcwd`'s test.
- **`PATTERN-test-only-code`** says a function only tests call lives in a `_test.go` file or a test-support package.
  `cmd/lym/testonlycode_linux_test.go` (tier 1, `//lym:guard`) enforces it.
  It runs `deadcode`'s analysis in process through `golang.org/x/tools`:
  rapid type analysis over SSA, rooted at every `main` package under `cmd/` and `tools/`,
  and at every function of a test-support package, so a function only `hubforge` calls counts as reached.

## Functions kept for tests of live code

These moved into `_test.go` files, `export_test.go` or a kit instead of being deleted,
because tests of live code compare against them or use them for setup.

References or oracles:

- `pattern.Check` and its helpers, the checker run over the live `PATTERN.md`.
- `planglyph.ValidateFormat`, `ValidateRework` and `Validate`, thin wrappers over the live validators.
- `preflight.Wired`, `shuttleengine.IsDraftHold`, `loomshed.ArchivedPlanWebsterDirs`
  and `tokencount`'s `RunTally.RoleTime`.
- `fabricengine`'s snapshot SHA helper, exposed to tests through `export_test.go`.

Setup helpers:

- `boardengine`'s `Board.Promote` and `Store.Promote`.
- `creelengine`'s `Post`, `PostWait` and `PostInterrupt`.
- `agentconfig.Roles`, `discussionparser.AppendDecision` and `gateslot.TemplateConfig`.
- `fabricengine`'s `commitWeft`, `WireJunctions`, `CodeLymLink` and `PushAnchored`.
- `websterengine`'s template accessors, `codexengine.New`, `preflightshed.NewPreflight` and `godocreflow`'s `reflowSource`.

## Design decisions

- **`gitkit` and `hubforge` count as test-support packages beside `internal/testkit`.**
  `gitkit` spawns git and `hubforge` builds hubs through `fabriccli`,
  and `PATTERN-test-isolation` bars both from a kit, so neither can move there.
- **References other packages' tests share stay in production, each named in the guard's allowlist.**
  A `_test.go` file cannot be imported by another package's tests.
  Each entry either reads its package's own state, such as `claudeengine.New`,
  or writes a record its package alone writes, such as `driftengine.UpdateScope` and `websterengine.WriteReport`.
  Every entry names the tests it serves, and an entry that matches no unreachable function fails the guard.
  The entries are `loomrecipe.RecipeEngines`, `shedrecipe.Names`, `driftengine.UpdateScope`,
  `fabricengine.RecordPushFailure`, three `fabricvocab` checks, `hibernateengine.TerminalPhases`,
  `seatengine.Prompts`, `shortcutcli.Load`, `claudeengine.New`, `slackengine.WriteSetup`
  and three `websterengine` functions.
- **The guard compiles on Linux only.**
  The analysis type-checks one build, and a Linux machine cannot type-check the Windows build of `quarry`'s cgo code.
  A function only a Windows file calls must sit in a Windows-only file, as `tmuxProcessName` now does.
  The pattern lists this and the other bounds of the analysis.
- **Safety checks before deleting.**
  `deadcode` ran rooted at `./cmd/...` and `./tools/...` under `GOOS=linux`.
  It could not run under `GOOS=windows`, where `quarry` needs a cgo cross toolchain,
  or under `GOOS=darwin`, where `internal/proc` has no implementation.
  In their place, every candidate's name was checked against every file only Windows or darwin compiles;
  that found `tmuxProcessName` alone.
  Each candidate's name was also checked for reflection, `go:linkname`, template function maps and string lookups,
  and none was reached that way.

## To check by hand

- `loomrecipe.RecipeEngines` and `shedrecipe.Names` in the guard's allowlist are pure oracles with no private state.
  They could move into `shedcheckkit` as `battenrecipe.RecipeEngines` did; decide whether they should.
- The `GOOS=windows` and `GOOS=darwin` deadcode runs did not happen; confirm no Windows-only or darwin-only file calls a deleted function.
- No regression check ran for this task.
