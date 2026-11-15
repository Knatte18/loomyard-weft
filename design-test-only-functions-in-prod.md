# Prod code holds test-only functions: 156 unreachable from main; add a PATTERN rule

`test-only-functions-in-prod` · Task · enhancement, pattern · draft

Imported from [issue #123](https://github.com/Knatte18/loomyard/issues/123).

## Seen

`deadcode ./cmd/lym` (golang.org/x/tools/cmd/deadcode) on `main` at `35f5f8670` reports 156 functions in production code that the `lym` binary never reaches from `main`.
The packages with most are `internal/fabricengine` (16), `internal/shedcheck` (9), `internal/websterengine` (8), `internal/loomcli` (6) and `internal/logger` (6).

A spot check shows most are not dead code but helpers that only tests call, yet they sit in non-test files:

- 54 of the 156 are `RunCLI` or `RunCLIIn` in the `*cli` packages; their callers are `_test.go` files.
- `batcher.ProfileWeights` and `batcher.PeakContext` are called only from `batcher`'s tests and their own file.
- `cmd/lym/main.go` has `run`, which repeats the body of `main` (build the root, set output, run, notify and sweep) and is called only by tests, so that logic exists twice.

The repository is about 376 000 lines of production Go and 546 000 of test Go; test-only helpers in production files add to the first figure without adding behaviour.

## Not known

Which of the 156 have no caller at all, tests included.
`deadcode -test` only roots the tests of the packages named, so each package needs its own run; this report did not do that.

## Idea

Add a PATTERN rule: a function that only tests call lives in a `_test.go` file or in `internal/testkit`, never in a production file.
Enforce it with a pattern test that runs `deadcode` per package, and fails on an unreachable exported or unexported function that a non-test file declares and only test files call.
Then move the existing 156 over, starting with `RunCLI` and `RunCLIIn`, and fold `cmd/lym`'s `run` into one shared function that `main` and the tests both call.
Run `deadcode` per package first to separate the truly dead functions from the test-only ones, and delete the dead ones.
