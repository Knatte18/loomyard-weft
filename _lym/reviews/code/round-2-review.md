---
verdict: BLOCKING
findings:
  - id: F15
    severity: BLOCKING
    class: design
    location: internal/loomcli/darnwiring.go:321
    summary: wireDarn renders and DarnTable requires the measuring call for every darn writer, so a hub whose darn-write tiers hold no {args} now halts every darn run at Preflight, the ungated runs included, whose prompt never names the call and whose verify runs that key fine.
    fix: "In internal/loomcli/darnwiring.go's darnTable closure, load the gate config and call gateCfg.MeasureCall(darnWriteGateKey) only when gateFixed is true, and leave MeasureCall empty otherwise. In internal/darnengine/spec.go DarnTable, drop {\"MeasureCall\", in.MeasureCall} from the unconditional required list and refuse an empty MeasureCall by name only when in.GateFixed is set; update the MeasureCall field comment to say only a GateFixed writer needs it. In internal/darnengine/spec_test.go TestDarnSpec_Refusals, make the empty MeasureCall row set GateFixed true, and add an assertion to the ungated TestDarnSpec row that a DarnInputs with MeasureCall empty builds. In internal/loomcli/wiring_darn_test.go's measuring-call subtest, add a row whose darn-write tier command holds no {args} with the fixer off, asserting the table builds."
  - id: F16
    severity: BLOCKING
    class: design
    location: internal/gatecli/trx_test.go:80
    summary: TestVSTestCounterReplacesAnInvocationsConsoleCounts and TestRemoveResultsDir pin behaviour TestRunTierReadsTheVSTestResultsFile already pins, against PATTERN-test-speed's rule that a test pins behaviour no other test covers.
    fix: "Delete TestVSTestCounterReplacesAnInvocationsConsoleCounts and TestRemoveResultsDir from internal/gatecli/trx_test.go. The row EachInvocationReplacesOnlyItsOwnConsoleCounts of TestRunTierReadsTheVSTestResultsFile in internal/gatecli/tierrun_spawn_test.go already asserts the per-invocation baseline, the replaced counts and both invocations' failures. Its row ACrashedTestHostNamesItsCauseAndNotAWarning asserts resultsFileLines' exact line. Every non-refusal row asserts each results directory is removed. Keep TestReadTRXResults, which alone pins summing several .trx files of one directory, skipping a non-.trx file and a missing directory. Drop the imports trx_test.go no longer uses."
  - id: F17
    severity: MEDIUM
    class: design
    carries: F4
    origin: assist:assist
    location: internal/gateslot/tiers.go:473
    summary: "MeasureCall still renders a call the gate refuses: for a key listing tier0 (command with {targets}) and tier1 (no target placeholder, covers: [0]), it adds the target word from tier0, but planTierRuns drops the covered tier0 and refuses explicit targets that no kept tier takes (internal/gatecli/tiers.go:147). It also checks {args} on covered tiers the call never runs."
    fix: "In Config.MeasureCall in internal/gateslot/tiers.go, first reduce gate.Tiers to the tiers the call runs: sort and compact them, then drop each tier that a higher listed tier's Covers holds, as gatecli's dropCovered does. Run the {args} check and the HoldsTargets check over that reduced list only. Update the doc comment and gateslot/doc.go's MeasureCall paragraph to say a covered tier counts for neither. Add a row to TestConfig_TierUse's MeasureCall table in internal/gateslot/config_test.go: tier0 `dotnet test {targets} {args}`, tier1 `dotnet test All.sln {args}` with Covers [0], a key listing [0, 1], want `lym gate test --gate <key> -- <args>`."
  - id: F18
    severity: LOW
    class: design
    carries: F7
    origin: assist:assist
    location: internal/gatecli/tierrun_spawn_test.go:264
    summary: TestRunTierReadsTheVSTestResultsFile builds its expected command words by calling vstestResultsArgs, the helper the code under test calls, so dropping the trx logger or --blame from that helper leaves the test green.
    fix: "In internal/gatecli/tierrun_spawn_test.go TestRunTierReadsTheVSTestResultsFile, replace `wantArgs = append(wantArgs, vstestResultsArgs(resultsDir)...)` with the literal words: `wantArgs = append(wantArgs, \"--results-directory\", resultsDir, \"--logger\", \"trx;LogFilePrefix=gate\", \"--blame\")`. Keep the following append of the call's own args."
prior:
  - id: F1
    status: resolved
  - id: F2
    status: resolved
    answer: "Settled by the added entry \"Drop the MeasureCall tier fallback\": MeasureCall returns Gate's refusal and renders no --tier call."
  - id: F3
    status: resolved
  - id: F4
    status: open
    answer: The no-{args}, no-tier0 and card-scope cases now refuse, but a covered target-taking tier still adds a target word the gate refuses; carried as F17.
  - id: F5
    status: resolved
  - id: F6
    status: resolved
  - id: F7
    status: open
    answer: The rows now pin the baseline, outcome mapping, fallbacks and passing run messages, but the gate's result words are asserted through the production helper; carried as F18.
  - id: F8
    status: resolved
  - id: F9
    status: resolved
  - id: F10
    status: resolved
  - id: F11
    status: resolved
  - id: F12
    status: resolved
  - id: F13
    status: resolved
  - id: F14
    status: resolved
---

### [BLOCKING] The measuring call halts darn runs that never use it

**Location:** internal/loomcli/darnwiring.go:321

**Issue:** The darnTable closure calls `gateCfg.MeasureCall(darnWriteGateKey)` whatever `gateFixed` says, and `DarnTable` lists `MeasureCall` among its unconditional required inputs (internal/darnengine/spec.go).
Since the round-1 F4 fix, `MeasureCall` refuses a darn-write key that lists a tier whose command holds no `{args}`.
Such a key is a valid config: `lym gate test --wait --gate darn-write` runs it with no words after `--`, so Darn-Write's verify passes on it.
It is also the shape docs/dotnet-hub.md recommended for tier0 before this diff, `dotnet test {targets} --no-build {filter}`, which the template's `darn-write: {tiers: [0]}` names.
On such a hub every darn run now fails when Preflight builds the table.
That includes a writer with no cold fixer, whose ungated Step 4 fragment never names `{{.measure_call}}`.
The parent's "Drop the MeasureCall tier fallback" entry rests on "a darn run whose hub cannot run the darn-write key fails its own verify anyway".
That premise holds for Gate's own refusals, not for this one, so the entry does not license halting these runs.
The regression sits at this changed line; the broken site is the Preflight table build.

### [BLOCKING] Two new tier0 tests pin what the tier1 results test already pins

**Location:** internal/gatecli/trx_test.go:80

**Issue:** Commit 6fc07cbe added `TestVSTestCounterReplacesAnInvocationsConsoleCounts` and `TestRemoveResultsDir` to answer the gate's reach report.
That report lists `TestRunTierReadsTheVSTestResultsFile` as "not judged, out of process", so it could not see that test's coverage.
Its assertions already cover both tests:
- The row `EachInvocationReplacesOnlyItsOwnConsoleCounts` runs two invocations whose console summaries disagree with their files. It asserts the literal summed counts and both failures, which is the baseline and replacement the counter test drives by hand.
- The crash row asserts the exact line `resultsFileLines` renders.
- Every row that runs asserts each results directory is gone after the run, which is all `TestRemoveResultsDir` checks apart from the trivial empty-path call.

PATTERN-test-speed requires that a test pin behaviour no other test covers.
`TestReadTRXResults` is no duplicate: it alone pins summing several `.trx` files of one directory, skipping a non-`.trx` file and a missing directory.

### [MEDIUM] Covered tiers can make the measuring call invalid

**Location:** internal/gateslot/tiers.go:473

**Issue:** `MeasureCall` adds `<its package or test project>` when any listed tier's command takes targets, and checks `{args}` on every listed tier.
`planTierRuns` first drops each tier that a higher listed tier covers (`dropCovered`, internal/gatecli/tiers.go:109).
It then refuses explicit targets when no kept tier takes them (internal/gatecli/tiers.go:147).
Take a key listing tier0 `dotnet test {targets} {args}` and tier1 `dotnet test All.sln {args}` with `covers: [0]`.
For it, MeasureCall renders a target word, and the gate refuses that call.
The doc's claim that "the call it returns is therefore one the hub's gate config can run" does not hold for that key.
This is the remainder of round 1's F4.

### [LOW] Expected result words repeat the implementation

**Location:** internal/gatecli/tierrun_spawn_test.go:264

**Issue:** The command under test gets its result words from `vstestResultsArgs`, and the test's expected argument list calls the same helper.
The fixture finds the results directory by `--results-directory` alone.
So removing `--logger trx;LogFilePrefix=gate` or `--blame` from the helper changes both sides alike, and every row still passes.
The rubric calls this a tautological expectation; round 1's F7 asked that the gate's words reaching the command be pinned.

## Focus verdicts

- `runInvocations`: the result words reach only a tier with `results: trx`, which load accepts only on a `vstest` tier whose command holds `{args}`. docs/dotnet-hub.md's timing tier sets no `results` and keeps its meaning. Not a defect.
- `Config.MeasureCall`: it renders no `--tier` fallback, per the parent's entry, so PATTERN-test-speed and PATTERN-gate-slot hold. A hub with no tier0 and a card-scope key are refused, and so is a tier without `{args}`; the wiring consequence of that refusal is F15. The covered-tier case is F17. A hub that declares no tier gets `--gate darn-write -- <args>`, which run.go accepts and runs as lints only, consistent with that hub's own verify.
- `envOverrideLine`: the line names each differing variable with the tier's value only, never the session's. It carries no role clause, prints once to stderr per tier run, whatever the copies, and heads the log of a run that was not refused. A `retryTier` rerun prints it once more with the rerun's own log, which matches its record. Not a defect.
- Test tiers: the spawning tests are tier1 in a `!windows` file with no run-time skip, and the TRX and counter tests are tier0 with no spawn. They pin the baseline across invocations, the outcome mapping, both console-count fallbacks and the call's words. The gate's own words are pinned only tautologically (F18), and two tier0 tests duplicate the tier1 test (F16).
- `trx.go`: only RunInfo messages of outcome Error become run messages. They become causes only on a non-zero exit. `runInvocations` flushes the counter's pending unterminated line before `replaceInvocation`, which the no-newline row pins. Not a defect.

## Rejected

- None of the assistant's findings is rejected: its covered-tier finding carries as F17 and its tautological-assertion finding as F18.
