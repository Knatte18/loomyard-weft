---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/gatecli/tierrun.go:487
    summary: "The gate prepends `--results-directory`, `--logger trx;LogFilePrefix=gate` and `--blame` to the `{args}` of every tier with `counter: vstest`, whatever program its command runs. This breaks the documented `dotnet run ... -- {args}` BenchmarkDotNet timing tier and changes what an existing config setting means, with no reinterpretation and a two-way merge-danger call."
    fix: "In internal/gatecli/tierrun.go runInvocations, stop deciding the injection from `counter.(*vstestCounter)` plus `{args}` alone. Preferred: an explicit opt-in tier setting declared in gateslot's Tier and config spec, such as `results: trx`. Gate the injection on it, and update docs/dotnet-hub.md's Tiers section and the recommended tier0 example to set it. Then a hub's existing tiers keep their meaning and no hubreconcile reinterpretation is needed. Only if the parent prefers changing the meaning of existing config does the change ship a hubreconcile reinterpretation per PATTERN-config-layers. In either case: update docs/dotnet-hub.md's timing-tier section to say that tier gets no result words; update the gatecli doc.go paragraph that opens 'A `vstest` invocation whose command holds `{args}`' and the runInvocations doc comment; and correct _lym/landing/summary.md's Merge danger, which must name the config-meaning change and the darn prompt's new gate.yaml read, and its Open risks."
    origin: lens:code-affected-set
  - id: F2
    severity: BLOCKING
    class: decision
    location: internal/gateslot/tiers.go:436
    summary: "MeasureCall's fallback renders `lym gate test --tier 0 ...` into the darn writer's prompt. PATTERN-test-speed says a site or stencil names a gate key, never a tier list. PATTERN-gate-slot calls gateslot.FailedTestsCall the one call a site spells with tiers. The board entry licenses the `--tier` fallback; per this review's rules the constraint wins over that license, and no Pattern-Departure trailer or declared PATTERN change covers the conflict."
    fix: "Leave the code in place and defer to the run's parent, which must record the call with `lym loom decision add`. Either declare a PATTERN change that admits Config.MeasureCall's `--tier` fallback as a second sanctioned tier-spelling call, then amend pattern/PATTERN-test-speed.md ('A site or stencil names a gate key...') and pattern/PATTERN-gate-slot.md ('the one call a site spells with tiers and targets') in the same branch. Or drop the fallback: when Gate refuses the key, render a measuring step that tells the writer to ask its parent for the run, and declare no tier."
    origin: handler
  - id: F3
    severity: BLOCKING
    class: design
    location: internal/gatecli/tierrun_test.go:453
    summary: "TestRunTierReadsTheVSTestResultsFile and TestRunTierNamesEnvThatDiffersFromTheSession call tiergate.Require(t, 0) but spawn `sh` (and `cp`) through runTier. PATTERN-test-speed keeps tier0 to no spawn and asks for the lowest tier that fits. They also skip at run time on Windows, where that PATTERN wants an OS-bound test in a file only that OS compiles."
    fix: "Move both tests into a new test file in internal/gatecli whose functions all call tiergate.Require(t, 1), with a `//go:build !windows` constraint, and delete their `runtime.GOOS == \"windows\"` t.Skip blocks. Leave the other tests of tierrun_test.go as they are."
    origin: assist:assist
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/gateslot/tiers.go:436
    summary: "MeasureCall can render a call the gate refuses, against its doc's claim that the call names only a form the config can run. It always appends `-- <args>`, which planTierRuns refuses for a tier whose command holds no {args} (internal/gatecli/tiers.go:116). Its fallback names tier0 when the hub declares tiers but no tier0. A key of scope card renders with no --card and no target, which gateKeyRefusal refuses (internal/gatecli/run.go:450)."
    fix: "In Config.MeasureCall, accept the key only when Gate accepts it, its scope is not card, and every tier it runs holds {args}. Otherwise fall back to the lowest declared, non-manual tier whose command holds {args}, adding the target word only when that tier holds targets. When no such tier exists, return a measuring step that tells the writer to ask its parent for the run, not a command. Update the doc comment and gateslot/doc.go to match. Add rows to TestConfig_TierUse/MeasureCall: tier0 without {args} beside tier1 with {args}; no tier0; a card-scope key; a key mixing a tier that holds targets with one that does not; and a fallback tier with no targets."
    origin: lens:code-correctness
  - id: F5
    severity: MEDIUM
    class: design
    location: internal/gatecli/tierrun.go:252
    summary: "envOverrideLine ends with roleClause, so an informational line on a run that may pass tells every non-prime session that it 'reports status: FAILED and the orch runs them'. A darn writer measuring under a deliberately different tier env is pushed to fail its run."
    fix: "In internal/gatecli/tierrun.go envOverrideLine, drop `roleClause` and its `%s` from the format string. Keep the 'read the result under those values' guidance, and name the tier env change as the prime's own step without the FAILED clause. Update the expected line in TestRunTierNamesEnvThatDiffersFromTheSession to match."
    origin: handler
  - id: F6
    severity: MEDIUM
    class: design
    location: internal/gatecli/tierrun.go:243
    summary: "envOverrideLine prints the session's own value of each differing variable to stderr and the tier log. A secret the session holds and the tier overrides, such as a token, then lands in the gate log and in the calling agent's transcript, although the child never receives it."
    fix: "In envOverrideLine, print each differing variable as `NAME=<tier value>` and leave out the session's value (drop the '(the session has %s)' part). The tier value comes from gate.yaml, which `lym config gate --print` already shows. Update the doc comment, gatecli doc.go's 'with both values' sentence, _lym/landing/summary.md's env section and Open risks, and the expected line in TestRunTierNamesEnvThatDiffersFromTheSession. Add an assertion there that the session's value appears in neither stderr nor the log."
    origin: assist:assist
  - id: F7
    severity: MEDIUM
    class: design
    location: internal/gatecli/tierrun_test.go:307
    summary: "TestRunTierReadsTheVSTestResultsFile pins one invocation with one valid file, so several documented behaviours go unchecked. One is the startInvocation baseline across invocations, the mutation survivors at counters.go:475-476. Others are the Error/Timeout/Aborted and skipped outcome mapping, the console-count fallback when no file or only a malformed one is read, and a passing run's run messages staying out of the causes. Nothing checks that the gate's `--logger` and `--blame` words reach the command either."
    fix: "Extend the table in TestRunTierReadsTheVSTestResultsFile, not a new test. Add a row with two invocations whose console summaries are parseable and disagree with their TRX files, asserting the summed literal totals and both invocations' failures. Add a row whose TRX holds Error, Timeout, Aborted and NotExecuted results, asserting failed 3 and skipped 1. Add a row that copies no file and one that copies a malformed file, each asserting the console counts. Add a passing row whose TRX holds a RunInfo, asserting empty stderr and no cause. Make the fixture command record its whole argument list, and assert it opens with the words vstestResultsArgs returns, followed by the call's own args."
    origin: lens:code-tests
  - id: F8
    severity: LOW
    class: design
    location: internal/gatecli/run.go:121
    summary: "Nothing pins `request.sessionEnv = os.Environ()`. The env-line test sets sessionEnv directly, and the integration tests strip GOFLAGS, so deleting the capture turns the env line off on the CLI path with every test green."
    fix: "In internal/gatecli/test_integration_test.go, add a case to an existing startGate test whose tier env sets a variable the gateRun env sets otherwise, such as a harmless MARK variable. Assert the env line on the gate's stderr and in the tier log, and add an equal-value row asserting no line."
    origin: lens:code-tests
  - id: F9
    severity: LOW
    class: design
    location: internal/loomcli/darnwiring.go:317
    summary: "No test asserts that the darn prompt's measuring call follows the hub's gate.yaml. TestDarnSpec feeds a constant MeasureCall, and the wiring tests only build the table, so replacing `gateCfg.MeasureCall(darnWriteGateKey)` with any literal passes."
    fix: "In internal/loomcli/wiring_darn_test.go, extend TestWire_DarnRecipe (or the existing table build it runs) with a gate.yaml fixture lacking the darn-write key. Assert that the built table's prompt holds the call MeasureCall renders for that config, as a literal string."
    origin: lens:code-tests
  - id: F10
    severity: LOW
    class: design
    location: internal/gatecli/tierrun.go:492
    summary: "The gate's result words go ahead of the call's own words unchecked. A call that passes its own `--results-directory`, or a trx logger with its own file name, can leave the gate's directory without a TRX file. The console counts then stand again, so the verdict can again depend on console text, against doc.go's 'A console logger among the call's own words so never changes the verdict'."
    fix: "In runInvocations, when run.args holds `--results-directory` or a `--logger` whose value starts with `trx`, refuse the call before rendering with a gate test line. The line names the conflicting word and the way forward of dropping it, since the gate owns those words. Add a row to TestRunTierReadsTheVSTestResultsFile for that refusal."
    origin: lens:code-correctness
  - id: F11
    severity: LOW
    class: design
    location: internal/gatecli/trx.go:505
    summary: "trxFile.RunMessages takes every RunInfo Text whatever its outcome, so a failed run lists warning-level messages, such as a filter matching no test in an earlier invocation, as failure causes. The doc says each such message is a test host crash."
    fix: "Decode RunInfo's `outcome` attribute in trx.go (a small struct with Outcome `xml:\"outcome,attr\"` and Text) and keep only messages whose outcome is Error. Add a Warning RunInfo to the crash row of TestRunTierReadsTheVSTestResultsFile and assert it is not in the causes."
    origin: lens:code-correctness
  - id: F12
    severity: LOW
    class: design
    location: internal/gatecli/tierrun.go:543
    summary: "replaceInvocation runs before the counter's pending, unterminated console line is handled. A last summary or Failed line without a newline is parsed later by Flush, or joined with the next invocation's first line, and adds console counts on top of the TRX results."
    fix: "In runInvocations, call counter.Flush() after runTiedChild returns and before readTRXResults/replaceInvocation for an invocation with a results directory, so its whole console output is consumed before the replacement."
    origin: assist:assist
  - id: F13
    severity: LOW
    class: scope
    location: internal/gatecli/doc.go:100
    summary: "A vstest tier whose command holds no {args} still logs no crash cause when its test host crashes. The board entry asks for the cause in the gate log with no condition. doc.go and the summary name only the console-count fallback for such a tier, not that it gets no crash cause."
    fix: "Add a sentence after 'An invocation that wrote no readable TRX file keeps the console counts, as does a command without `{args}`.' in internal/gatecli/doc.go stating that such a tier gets no results file and so no crash cause, and that adding {args} to its command gives both. Say the same in _lym/landing/summary.md's TRX section."
    origin: lens:code-scope
  - id: F14
    severity: NIT
    class: design
    location: internal/gatecli/tierrun.go:138
    summary: "Under --copies N each copy's runTierCommands prints the env line, so stderr carries the same line N times. On Windows the session lookup is case-sensitive while environment names are not."
    fix: "Compute the env line once in runTierChecked, before choosing runTierCopies or runTierCommands, and print it there; drop the print from runTierCommands. In envOverrideLine, key sessionValues by strings.ToUpper(name) when runtime.GOOS is windows."
    origin: lens:code-correctness
---

### [BLOCKING] Result words reach every vstest tier with {args}, the documented benchmark tier included

**Location:** internal/gatecli/tierrun.go:487

**Issue:** The injection condition is `counter.(*vstestCounter)` and `{args}` in the command, nothing else (tierrun.go:486-493).
docs/dotnet-hub.md:145 recommends a timing tier `dotnet run -c Release --project NORCE.Models.Benchmarks -- {args}` with `counter: vstest`.
That BenchmarkDotNet program now gets `--results-directory`, `--logger trx;...` and `--blame`.
The setting `counter`, which says how output is counted, now also changes the command's arguments.
That changes what existing hub config means, and PATTERN-config-layers asks such a change to ship a reinterpretation.
The summary's `two-way` door and `gate` blast radius name neither.

### [BLOCKING] The measuring call's `--tier 0` fallback conflicts with the PATTERN gate-key rules

**Location:** internal/gateslot/tiers.go:436

**Issue:** When Gate refuses the key, MeasureCall renders `lym gate test --tier 0 ...` (tiers.go:442-443), and the darn wiring fills it into the stencil's `measure_call` marker.
pattern/PATTERN-test-speed.md, "Tests run through the gate", says "A site or stencil names a gate key, never a tier list".
pattern/PATTERN-gate-slot.md calls `gateslot.FailedTestsCall` "the one call a site spells with tiers and targets".
The board entry's #156 fix offers "fall back to the `--tier` form", so the code follows the answer key.
This review's constraints section says a constraint wins over the answer key when they conflict, and no `Pattern-Departure:` trailer or declared PATTERN change settles it.
Only the parent can choose between changing the PATTERN text and dropping the fallback, so the class is `decision`.

### [BLOCKING] The two new shell-spawning tests sit in tier0

**Location:** internal/gatecli/tierrun_test.go:453

**Issue:** Both new tests open with `tiergate.Require(t, 0)` and run `runTier`, which starts the platform shell through `runTiedChild`; the TRX fixture also runs `cp`.
PATTERN-test-speed's tier table gives tier0 "nothing: offline, no spawn", and its entry asks for the lowest tier that fits.
Its bound says the static scan cannot see a spawn through a production helper and leaves that line to review.
Both tests also `t.Skip` on Windows, where the PATTERN's "No test skips by design" wants a file only POSIX compiles.
The file's older tests follow the same shape, but that does not license new ones.

### [MEDIUM] MeasureCall can still render a refused call

**Location:** internal/gateslot/tiers.go:436

**Issue:** `planTierRuns` refuses words after `--` for a tier whose command holds no `{args}` (internal/gatecli/tiers.go:116-117), and MeasureCall always appends `-- <args>`.
Its fallback names tier0 even when Gate refused because the hub declares no tier0 (tiers.go:393-394).
A card-scope key passes Gate, but its rendered call carries neither `--card` nor a target when no tier takes targets, and gateKeyRefusal then refuses it (run.go:450-451).
In each case the prompt names a form the hub's config refuses, which is the defect #156 and #162 report.

### [MEDIUM] The env notice carries the role clause's FAILED instruction

**Location:** internal/gatecli/tierrun.go:252

**Issue:** The line ends with `roleClause`, which reads "a session that is not the prime takes none of these steps; it reports status: FAILED and the orch runs them".
The line is printed on passing runs too, and its first step is reading the result under the tier's values.
A non-prime agent that measures through the gate, the case #157 describes, is told to report FAILED for a notice.

### [MEDIUM] The env notice prints the session's value

**Location:** internal/gatecli/tierrun.go:243

**Issue:** Each differing variable is printed as `NAME=<tier value> (the session has <session value>)`, to stderr and the tier log.
The session's value never reaches the child, yet it now lands in the gate log and the calling agent's transcript.
The summary names this as an open risk without fixing it.
The tier's value alone tells the agent what it measured under.

### [MEDIUM] The TRX test leaves the baseline, the outcome mapping and the fallbacks unchecked

**Location:** internal/gatecli/tierrun_test.go:307

**Issue:** Every row uses `invocations: []invocation{{dir: dir}}` and copies one valid fixture, so dropping `startInvocation`'s baseline (the mutation survivors at counters.go:475-476) passes.
So do mapping Error, Timeout or Aborted to skipped, and calling replaceInvocation on an unread directory.
The fixture command reads only `--results-directory` and ignores `--logger` and `--blame`, so removing either word from vstestResultsArgs passes too.
No row has a passing run whose TRX holds a run message.

### [LOW] The session-env capture is unpinned

**Location:** internal/gatecli/run.go:121

**Issue:** TestRunTierNamesEnvThatDiffersFromTheSession builds `testRequest{sessionEnv: session}` itself.
The integration harness now drops GOFLAGS from the gate it starts (test_integration_test.go:403), so no CLI-level test sees an env line.

### [LOW] The darn wiring's use of the gate config is unpinned

**Location:** internal/loomcli/darnwiring.go:317

**Issue:** DarnTable's tests fill `MeasureCall` with a constant, and the wiring tests only check that the table builds.
Nothing ties the prompt's call to the hub's gate.yaml.

### [LOW] Caller-supplied result options can take the TRX file away from the gate

**Location:** internal/gatecli/tierrun.go:492

**Issue:** `args = append(vstestResultsArgs(dir), run.args...)` passes a caller's own `--results-directory` or trx logger after the gate's.
When the TRX file then lands elsewhere, readTRXResults reads nothing, and the console counts the #163 report distrusted stand again.

### [LOW] Warning-level run messages become failure causes

**Location:** internal/gatecli/trx.go:505

**Issue:** `RunMessages []string \`xml:"ResultSummary>RunInfos>RunInfo>Text"\`` collects every RunInfo, and runTierCommands prints each one as a cause on any failed run.
The gatecli doc says such a message is a test host crash.

### [LOW] The pending console line is read after replacement

**Location:** internal/gatecli/tierrun.go:543

**Issue:** lineReader keeps an unterminated last line in `pending`, and Flush runs only once every invocation has ended (tierrun.go:147).
The replacement for an invocation therefore happens before its last console line is parsed.

### [LOW] Tiers without {args} get no crash cause, and the docs do not say so

**Location:** internal/gatecli/doc.go:100

**Issue:** For a vstest tier whose command holds no `{args}`, resultsFileLines returns nothing, so a crashed host still leaves no cause in the gate log.
The doc and the summary name the console-count fallback for such a tier but not this gap.

### [NIT] The env line repeats under --copies

**Location:** internal/gatecli/tierrun.go:138

**Issue:** runTierCopies calls runTierCommands once per copy (copies.go:90-95), and each copy prints the line to the shared stderr.

## Rejected

- Correctness lens F5, first half, that the printed GOFLAGS lacks the gate's `-count=1`: the line names the tier's configured values against the session's, and `-count=1` is the gate's own flag on every tier, not an override the session could differ on.
- Tests lens NIT on a passing run with a run message: folded into F7.
- Tests lens LOW on MeasureCall's mixed-tier and target-free fallback rows: folded into F4's fix.
- Assistant F1's call for a config reinterpretation: kept within F1 as one of two routes, since an opt-in tier setting avoids changing an existing meaning.
- Assistant F5: merged into F10.
- Assistant F7 and F8: merged into F7.
- Assistant F9: merged into F8.
- Assistant F3 at BLOCKING: kept as F6 at MEDIUM, since the tier value comes from gate.yaml and only the session value is at risk.
- Assistant F4: merged into F4.
- Affected-set lens F2, the PATTERN text not updated: merged into F2, raised to a BLOCKING `decision` because a constraint conflict is the parent's to settle.
- Mutation survivor at tierrun.go:570, the negated condition in removeResultsDir: it flips only a warning log line, and every row already asserts the directory is gone.

## Focus checks without a finding

- Root-Cause trailers: each of the three feature commits names the defect its diff removes. The two `fix:` commits repair the branch's own work.
- Red-first waiver: the added entry waives the red-first gate for the whole run, and its rationale names only the stencil file. The diff relies on it for nothing else, so there is no finding.
- Stencil: the gated fragment keeps every rule it carried. Only the call and "every tier the key lists" → "every tier the call runs" change, and the latter follows the new call.
- The unknown-key refusal lists the declared keys sorted, through gateslot's sortedKeys.
- The results directory is removed on the normal, render-error, start-failure, memory-cap and timeout paths.
