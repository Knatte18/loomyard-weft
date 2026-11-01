# The gate reports every failure at once, lint included, in one result

A failing `lym gate test` call now closes with one failure report on stderr, listing every failure the call found.
An agent can read the whole report in one tool result and fix every failure before it runs the gate again.

## What changes

- **Failure report** (`internal/gatecli/failurereport.go`).
  Before the closing line of failed runs, a failing call prints `gate test: failure report, every failure of this call:`.
  The report then names each failed tier, vet, guard or build run with its outcome and log path.
  Under each run it lists every failed test as its package and test path, and every package that failed outside any test, such as a build error or a panic in `TestMain`, as its package.
  Each failure shows up to `failureLines` lines it printed, such as the got and want of an error or the first frames of a panic.
  A bare file name a test printed (`p_test.go:6:`) gets its package's worktree-relative directory (`report/p/p_test.go:6:`).
  A run with no parsed failure shows the verb's own cause line, such as a timeout or a missing build tag, and the first lines of output no test event carried, such as `go vet` errors.
  The report closes with the files the change touches, the diff from the merge base with the pair's parent branch, or from HEAD outside a hub.
  It is bounded by line count, line length and the number of files it names.
- **Counter** (`counters.go`).
  The `go-json` counter now keeps the first lines of each failed test, not only the first one.
  It also records failed packages that no failed test explains, from their package-level output and their `build-output` events, and keeps the non-JSON lines.
  The run record's `failures` stays as it was: each failed test with its first line.
- **Lint steps in the gate** (`gateslot.GateKey.Lints`, `runLints`).
  A `gate.yaml` gate key may list `lints: [comments, pattern]`.
  `lym gate test --gate <key>` runs those lints after its tiers, through the same `commentlint.Lint` and `patternlint.Lint` as `lym loom lint-comments` and `lint-pattern`.
  Each lint compares the working tree against HEAD.
  A lint finding fails the call with exit 1, appears in the report with its way forward, and is named in the closing line as `lint <name> (N findings)`.
  An unknown lint name is refused when the config loads.
  The template's `card` key lists both lints, so Webster's card gate is now one call instead of three steps (`websterengine.cardGateCommand`).
- **Bounded replay** (`detachfront.go`).
  A capped agent call, which joins a detached runner, now prints the runner's stdout only up to `replayStdoutBytes`.
  A longer stdout is named on stderr with its size and path, and its last line is still printed when that line is a JSON envelope, such as a refusal.
  The stderr, with the tier summary lines and the failure report, is printed whole.
- **Implementer prompt**.
  `webster-body-implementer.md` tells the fork to fix every failure the report lists before it runs the gate again.

## Design decisions

- The lints are a per-key config list, not a hard-coded step, following `PATTERN-no-hard-coded-choice`.
  The template's `card` key carries them, so every hub keeps the two lints its card gate ran before.
  The layer merge works per key, so a hub that tunes only the `card` key's tiers keeps the template's lints; `TestLoadConfig` pins this.
- In-process runs keep printing the tiers' full stdout: `--wait` calls, verify sites, `RunGate` and card-verify lines.
  The verify gates parse that output (`websterengine.ParseVerifyFailures`, `loomshed.verifyFailureFindings`), and their logs are unchanged.
  Only the capped front's replay is bounded.
- The lints run only under `--gate`, after every tier, and not after a halted call.
  A call made with manual flags runs no lint.
- The Darn-Write and Code-Burler-Fix gates keep their own lints from the merge base in `loomshed`; the gate's lint step compares against HEAD only, as the card gate's steps did.

## Docs

The `gatecli` package doc gains the Lint steps and Failure report sections, and the counter and capped-call paragraphs now describe the new behavior.
`gateslot`'s doc and template describe `lints`.
`docs/overview.md`, `PATTERN-gate-slot`'s self-check pairs, the plan spec, `loomshed`'s doc and the code-review rubric now name the gate key's lint step where they named the card gate's separate lint commands.

## Tests

- `TestCounters` adds rows for multi-line failures, the line cap, a failed build and a panic outside a test.
- `failurereport_test.go` covers rendering and its bounds, lint, build and tier failed runs, and the stdout replay bound.
- `TestGateTest_Scenario` adds two rows:
  - a real `go test` module whose report names a failed test with its qualified `file:line` and values, a broken package, the log and the touched files;
  - a gate key whose `comments` lint fails the call and lists the finding.
