# Remove test-only functions from production code and add a PATTERN rule against them

`test-only-functions-in-prod` · Task · enhancement, pattern, testkit, cli · running · Darn-Write

Imported from [issue #123](https://github.com/Knatte18/loomyard/issues/123).

## Seen

`deadcode ./cmd/lym` (golang.org/x/tools/cmd/deadcode) on `main` at `35f5f8670` reports 156 functions in production code that the `lym` binary never reaches from `main`.
54 of them are `RunCLI` or `RunCLIIn` in the `*cli` packages.
Each is a thin wrapper, `clihelp.Execute(Command(), out, args)` or `clihelp.ExecuteIn(...)`, that only tests call, often from other packages' tests.
Others, such as `batcher.ProfileWeights` and `batcher.PeakContext`, are logic only tests call.
`cmd/lym/main.go` has `run`, which repeats the body of `main` and is called only by tests.

## What to do (operator, 2026-11-15)

1. **`RunCLI` and `RunCLIIn`:** replace every call with a direct `clihelp.Execute(<pkg>.Command(), ...)` or `clihelp.ExecuteIn(...)`, then delete the wrappers.
   The tests then run the same code they ran before.
2. **Functions with no production caller:** delete each, together with the tests that test only it.
   Keep one, and move it into a `_test.go` file, when it is a reference or oracle that tests of live code compare against; say which in the change description.
3. **`cmd/lym`'s `run`:** fold it and `main` into one function both call.
4. **The rule:** add a PATTERN entry: a function only tests call lives in a `_test.go` file or in `internal/testkit`, never in a production file.
   Enforce it with a pattern test; a `deadcode` run that fails on a new such function is one way, and the Discussion of the run settles the way.

## Safety conditions (operator, 2026-11-15)

A function is deleted only when every one of these holds:

- `deadcode` roots every `main` package of the module, `cmd/...` and `tools/...` both, not `cmd/lym` alone.
- It is unreachable under `GOOS=linux`, `GOOS=windows` and `GOOS=darwin` alike, since build-tagged files such as `internal/fslink`'s Windows junctions call code a Linux analysis cannot see.
- No reflection, `go:linkname`, template function map or lookup by name reaches it; grep its name before deleting.
- The gate is green up to tier 2 after the change.
