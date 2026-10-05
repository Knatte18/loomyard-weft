# Measure the test suite, speed up integration, split four tiers

The Go test suite grew past what its structure supports: thousands of top-level tests, an integration tier near a minute, test runs that leave dead tmux socket files behind, and a `smoke` tag that mixed tmux-only tests with tests that spawn real LLM sessions.
This change measures the suite per test, removes real-time waits, stops the socket leak, splits the tags into four tiers and consolidates PATTERN's Testing section.
It deletes no tests: `test-suite-prune-core` and `test-suite-prune-rest` consume the new tool and report for that.

## What changes

**Measurement.**
`cmd/testtiming` reports per-package top-level test count and serial time, and takes `-tags` to run under any tag set.
A new `-redundancy` mode builds one coverage-instrumented test binary per package, runs each top-level test alone and compares covered blocks module-wide.
Per package it reports candidates with their covering tests, a removable set (a greedy pass, slowest first, so a mutually redundant pair is never removed together) and a "no coverage" list.
A test that spawns a subprocess is found by a static scan of the package's test sources and listed under "no coverage", never as redundant; the scan over-approximates, so a test it cannot classify counts as spawning.
The generated report is committed as `docs/benchmarks/test-redundancy.md`.

**Speed.**
The reed watchdog discovery interval, `reapExitTimeout` and verifyrun's `waitDelay` become per-instance seams that tests shorten; production values are unchanged.
`verifytree`'s integration test runs `true` instead of `sleep 5`.
fabricengine's serial integration tests run under `t.Parallel`, except where a test touches process-global state, which its comment names.
`bareVocabularyTokenIn` skips its regex when the text holds neither `weft` nor `warp`.
`internal/gitrepo/internal/gitoracle`'s tests run under the hermetic git environment.

**Socket leak.**
Tests that start a tmux server take their socket from `tmuxkit`, which removes the socket file after `kill-server`; `tmuxkit.Main`'s sweep removes the files it kills.
The ad hoc key schemes in the reedengine contract tests and the reedcli attach and dotfill harnesses moved onto it.
In production, reed's teardown now removes the socket file of the server it just killed (`internal/reedengine/socketfile.go`).

**Four tiers.**

| Tier | Tag | Needs |
|---|---|---|
| 1 | none | nothing outside the process |
| 2 | `integration` | git, processes, filesystem links, the built `lyx`; no tmux server |
| 3 | `tmux` | a live tmux server |
| 4 | `llm` | a real LLM session |

The `smoke` tag is retired.
Every former `smoke` file moved to `tmux` or `llm`, `integration` files that start a tmux server moved to `tmux`, and files that spawn an LLM moved to `llm`.
Tags do not nest.
A new kit `internal/testkit/llmkit` holds the one helper that locates the LLM binary and skips when it is absent, and a guard test in `cmd/lyx` (`llmtier_test.go`) fails when a non-`llm` file references `llmkit` or any file outside it looks up an LLM binary.

**PATTERN.**
The Testing section now has five entries: `PATTERN-test-economy`, `PATTERN-test-speed` (absorbs tier purity, time seam and parallel), `PATTERN-test-isolation` (absorbs hermetic git and tmux test isolation, and carries the socket rule), `PATTERN-testkit` (gains the bounded `llmkit` `os/exec` exemption) and `PATTERN-hubforge-fixtures`.
Enforcement tests keep their checks and name the new IDs; every other reference to a retired ID is rewritten.

**Docs.**
`docs/benchmarks/running-tests.md`, the verify-model section of `contracts/specs/loom-plan-spec.md`, and the verify text in `loom-template-plan.md` and `loom-rubric-plan-review.md` describe the four tiers: `llm` is compiled (`go vet -tags llm`), never run, and `tmux` is a token-free tier a plan verify may run.

## Measurements

Full tables and the slowest-tests lists are in `docs/benchmarks/test-suite-timing.md`.
Medians of three runs on the same machine:

| State | Tier 1 wall | Integration wall |
|---|---|---|
| Before | 9.02 s | 57.05 s |
| After speed fixes, same pre-retag test set | 5.97 s | 30.41 s |
| After retag | 8.38 s | 34.00 s |

The first two rows run the same 5593 integration tests, so the integration drop comes from the speed fixes alone.
The load average was higher during the after-speed run, which works against it.
The Tier 1 drop was not isolated and Tier 1 wall time moves with machine load.
The after-retag row is not comparable with the first two: tmux-starting files left the integration tier, which now has 5533 tests, and the new `tmux` tier takes 231.56 s.

## Design decisions to check

- Redundancy is judged on coverage blocks, which do not see assertions: a candidate may pin an error message or output shape no other test pins, so the prune tasks must check each before deleting.
- Seams are injected per-instance fields or options, never package-level variables, so tests that use them can still call `t.Parallel`.
- The production socket removal in reed is bounded: it removes only the file for the `-L` key it just killed, only when the path is a socket, no server answers on it and its file identity is unchanged between check and removal.
  It never globs the socket directory.
  A server restarting on the same key in that window is the accepted residual race.
- Socket files already in `/tmp/tmux-$UID/` from earlier runs are not cleaned.
- A helper file shared by tests of two tiers whose helpers spawn a process or tmux server carries the disjunction of those tiers (for example `//go:build tmux || llm`), so it compiles under each tag alone; tier purity and `PATTERN-testkit` leave no other home for the reedcli and shuttlecli harnesses.
- `cardGateCommand` is unchanged and still runs `-tags integration`, so Webster's card gates no longer run the `tmux` tier until note `test-tiers` decides gate selection.
  Files moved to `llm` leave every automatic run; they are compiled by `go vet -tags llm ./...` and run by hand.
- The guard sees static shape only: a `tmux`-tier test that drives `lyx` into spawning an LLM without calling the helper is caught by review.

## To check by hand

- `go test -tags tmux ./...` was not run clean in the run records.
  The post-retag timing recorded `reedcli.TestSmokeDotFillCrossClientControl` failing on every `-tags tmux` run: tmux 3.6 refused with "no space for new pane" on the 8-row toucher pane.
  The test already failed at the base commit.
  A later fix (`cddbf1e7a`) makes it wait on `DOTFILL-MARKER-ALPHA`, the strand an 8-row pane can render, and its author ran only `go test -tags tmux ./internal/reedcli` and `-tags integration ./internal/reedcli`.
  Run the full `-tags tmux ./...` and confirm this test passes on your tmux version.
- A full run of every tier but `llm` leaves no new file in `/tmp/tmux-$UID/`; this was not re-verified on the final tree.
- Run `go vet` under `integration`, `tmux` and `llm` each alone, and `go test ./...`.
