# Card 2 — creel-suite-subcommand

**Create:**
- `plan:tools/sandbox#creelSandboxSuiteMD` -> `var creelSandboxSuiteMD string`
- `plan:tools/sandbox#creelSuite` -> `var creelSuite suiteSpec`

**Edit:**
- `tools/sandbox#suiteSpec`
- `tools/sandbox#run`
- `tools/sandbox#TestRunSuite_PerSuiteSpec`
- `tools/sandbox#TestSuiteSpecs_ReedTeardownFlag`
- `tools/sandbox#TestRun_SuiteSubcommands`
- `tools/sandbox#TestRunSuite_ReedTeardown`

**Uses:**
- `tools/sandbox#runSuite`
- `tools/sandbox#shuttleSuite`
- `tools/sandbox#stubSuiteSeams`
- `tools/sandbox#makeHubRepo`
- `tools/sandbox#captureStderr`

**Intent:** Add the `creel-suite` subcommand of the sandbox tool, running the creel suite file card 1 wrote through the shared `runSuite` (D4).

In `tools/sandbox/suite.go`, `creelSandboxSuiteMD` is a `//go:embed SANDBOX-CREEL-SUITE.md` string beside the other embeds, and `creelSuite` is a `suiteSpec` value initialised like `shuttleSuite`: `fileName: "SANDBOX-CREEL-SUITE.md"`, `doc: creelSandboxSuiteMD`, `instruction: "Read ./SANDBOX-CREEL-SUITE.md and follow the instructions in it exactly."`, `reedTeardown: true`.
Its doc comment names it the SANDBOX-CREEL-SUITE spec exercising `lyx creel`'s black-box scenarios, and says it sets `reedTeardown` because its scenarios add reed strands, whose tmux server would otherwise block the next `build -reset`.

In `tools/sandbox/main.go`, `run` gains a `case "creel-suite":` after `"shuttle-suite"`, a copy of the shuttle case with its own `flag.NewFlagSet("sandbox creel-suite", flag.ContinueOnError)`, the same `-claude` and `-prompt` flags, and `runSuite(absParent, *claudeFlag, *promptFlag, creelSuite)`, printing `sandbox: <err>` and returning 1 on an error.
The subcommand adds no mechanic and opens no new route: it runs only `runSuite` with a fixed spec, so it can write only `SANDBOX-CREEL-SUITE.md` and remove a stale `sandbox-report.json` inside the hub's warp repo, append those two names to that repo's `.git/info/exclude`, launch the agent there and run `lyx reed down` after it; an absent hub is refused by `runSuite`'s existing `run sandbox/build.cmd first` error before anything is written or launched.

The file comments name no tally (Notes for the plan writer): `main.go`'s header lists `creel-suite` (running the embedded SANDBOX-CREEL-SUITE agent) among the subcommands, drops "nine subcommands" for wording with no count, and drops "matching the other six" for "matching the other suites"; its last sentence's slash list of flag-parsing subcommands gains `creel-suite`.
`suite.go`'s header names `"sandbox creel-suite"` and the creel template among the subcommands and suites, and "The six suites share" becomes "The suites share".
`suiteSpec`'s doc comment drops "the six supported suites (main, reed, shuttle, burler, webster, and fabric)" for "every supported suite".

Tests, each a row in an existing table, no new top-level test:

- `TestRun_SuiteSubcommands` gains the row `{"creel-suite", creelSuite}`, covering dispatch with the default instruction, `-claude`/`-prompt` routed after the token, and the absent-hub exit.
  Its `absent Hub exits non-zero` subtest stubs the seams through `stubSuiteSeams` with a launch function that calls `t.Error` when invoked, a guard so that a regression launching the agent before the hub check fails the row instead of starting a real `claude`; `TestRunSuite_HubAbsent` stays the test of the hub check itself.
  The same subtest runs `run` under `captureStderr` and asserts the captured stderr contains `sandbox/build.cmd`, so each row pins that its subcommand surfaces the build-first refusal, not only a non-zero exit.
  A new subtest, `unknown flag exits non-zero`, builds a hub with `makeHubRepo`, stubs the seams with a `t.Error` launch function, runs `run` with the token followed by `-bogus` under `captureStderr`, and asserts a non-zero exit and no launch.
  This reaches each suite case's flag-parse failure exit; the hub is present so that an ignored parse error would launch the agent and fail the row, where an absent hub would return non-zero anyway.
  The test's doc comment names both subtests.
- `TestRunSuite_PerSuiteSpec` gains `creelSuite` in its `suites` list and the row `{"creel", creelSuite}` in its table, covering the default instruction, the scheme file written with the fingerprint header and embedded body, no other suite's file written, both names git-excluded, the stale report removed and the `-prompt` override.
  Adding `creelSuite` to `suites` also makes every other row assert it writes no `SANDBOX-CREEL-SUITE.md`.
  The `default run` subtest also reads `tt.spec.fileName` from the package directory (`os.ReadFile(tt.spec.fileName)`, the test's working directory being `tools/sandbox`) and asserts it equals `tt.spec.doc`, so a spec embedding another suite's file fails its row without pinning any suite text; the existing `tt.spec.doc` check compares the spec against itself and cannot catch that.
- `TestSuiteSpecs_ReedTeardownFlag` gains `creelSuite` in its live-reed list, so a creel spec with `reedTeardown` false fails it.
- `TestRunSuite_ReedTeardown` gains the row `{"creel suite tears down once after the agent", creelSuite, 0, nil, 1}`, pinning that a creel run calls `reedDown` once, after the agent session, with the repo directory and the resolved lyx.

`cmd/lyx/testdata/test-budget.yaml` is untouched, since no top-level test is added.

**ImpactSummary:** one new dispatch case and spec in `tools/sandbox`, a `main` package no other package imports; three existing table tests gain rows and one subtest gains a seam stub, with no caller outside the package.
