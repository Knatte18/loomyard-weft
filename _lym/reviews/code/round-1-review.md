---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: lens:code-affected-set
    location: internal/gatecli/keynames.go#tierKeyName
    summary: The new tier0 and tier-N refusals build their key name from a repeated "tiers." literal, which breaks PATTERN-config-key-names and PATTERN-code-conventions
    fix: "In internal/gatecli/keynames.go, add a named constant tiersKey = \"tiers\" to the key-path const block, add tierKey(tier string) string that returns keyName(tiersKey + \".\" + tier), and make tierKeyName build its path from tiersKey. Use tierKey(tierName(0)) at internal/gatecli/tiers.go planTierRuns, the no-tier0 refusal. In internal/gateslot/keynames.go, add the same tiersKey constant and a tierKeyName(tier string) helper over keyName, and use it in Config.Tier and Config.Gate at internal/gateslot/tiers.go, the two refusals the diff changed."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/tierrun.go#withTimeoutMin
    summary: TestWithTimeoutMin cannot tell the wall-clock term from the monotonic one, so a mutant that drops the wall clock, the core of the #215/#216 fix, passes
    fix: "Give withTimeoutMin an elapsed seam in place of now: a func elapsed() time.Duration built in production from a start time and time.Now as max(time.Since(start), time.Now().Round(0).Sub(start.Round(0))), or two injected readings, one monotonic and one wall. In TestWithTimeoutMin (internal/gatecli/tierrun_test.go), add a row whose monotonic elapsed stays at 1 minute while the wall elapsed reads 21 hours against a 35-minute timeout, and assert the cause errTimeoutMinPassed."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/tiers.go#withOtherTiers
    summary: No test asserts the otherTiers that planTierRuns fills, so dropping withOtherTiers on either return path passes every test while the notice stops naming tiers
    fix: "In TestPlanTierRuns (internal/gatecli/tiers_test.go), add rows that assert each run's otherTiers: a --gate key running tier0 with tier1 and tier3 declared wants [1 3]; a tier0 that covers tier1 wants [3]; and one row through the guard return path (request.guards under the go scoper) wants the same list."
  - id: F4
    severity: MEDIUM
    class: design
    origin: handler
    location: contracts/stencils/burler/burler-template-fixer.md:45
    summary: The new no-gate paragraph tells the fixer to defer on any refusal of lym gate test, so a refusal whose way forward is the fixer's own corrected call is deferred instead of followed
    fix: "Narrow the paragraph to refusals whose way forward is a config change from the prime. Replace its first sentence with: When `lym gate test` refuses to run any tier and its way forward is a change to the hub's gate config, as on a hub that declares no tier0, the round has no gate. Add a sentence after the paragraph: A refusal whose way forward changes your own call, such as dropping a word after `--`, is no such case: follow its way forward and run the gate again."
  - id: F5
    severity: LOW
    class: scope
    origin: lens:code-scope
    location: internal/gatecli/copies.go:66
    summary: A --copies tier's shared prebuild is still bounded by context.WithTimeout on the monotonic clock, though doc.go now says the tier's timeout_min bounds the prebuild and counts wall-clock time
    fix: "In runTierCopies, replace context.WithTimeout(ctx, time.Duration(run.tier.TimeoutMin)*request.timeoutUnit) with withTimeoutMin(ctx, time.Duration(run.tier.TimeoutMin)*request.timeoutUnit, time.Now, time.After), as runTierCommands does."
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gateslot/config_test.go:883
    summary: The Config.Tier and Config.Gate refusals now name gate.tiers.tier<N>, but their test rows assert only the substring before it, which the mutation survivor at gateslot/tiers.go:415 confirms
    fix: "In internal/gateslot/config_test.go, make the rows at lines 883 and 918 assert the whole refusal text Config.Tier and Config.Gate return, including the key gate.tiers.tier0, written out as a literal."
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/counters_test.go#TestCounters
    summary: No row covers a failed test's block closing at a Total tests line, the clause the diff adds to observeFailure
    fix: "Add a vstest row to TestCounters whose writes are a Failed <name> [ms] block with an Error Message line followed directly by Total tests: 1 and Failed: 1 lines and a Total time line, asserting the failure's text holds no summary line and the counts are discovered 1, failed 1."
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/tiers_unix_test.go#TestRenderTierCommand
    summary: No row checks that a caller filter still stands when the class terms are dropped past greenCommandBudget
    fix: "Add a TestRenderTierCommand row with command t {filter} {args}, filter Category!=Slow, terms past greenCommandBudget and args --filter Name=X, wanting t --filter '(Category!=Slow)&(Name=X)' followed by the empty args."
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/gatecli/tierrun_test.go#TestWithTimeoutMin
    summary: The redundancy report lists TestWithTimeoutMin as contained by TestRunBuildTimeout, but it pins the poll lengths and the cause on a fake clock, which the covering test never checks
    fix: "Add the line //testtiming:keep the poll lengths and the cause of the timeout_min bound on a fake clock directly above func TestWithTimeoutMin."
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/gatecli/tiers.go#planTierRuns
    summary: The doc comment of planTierRuns and the package doc's list of planning refusals leave out the new refusal of a caller --filter for a command with its own --filter
    fix: "Add the refusal to the list in planTierRuns's doc comment, the sentence that begins It refuses, naming `lym config gate`, as: a caller --filter for a command that spells its own --filter outside {filter}. Add the same item to the list of planning refusals in internal/gatecli/doc.go, the paragraph that opens Once the config is loaded, planning refuses these."
  - id: F11
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/counters.go#observeSummary
    summary: A Total tests summary ends at the first line that is no outcome line, so a line interleaved from another project or from stderr drops that summary's Passed, Failed and Skipped counts
    fix: "In observeSummary, end the open summary only at a line matching the Total time line (add a vstestTotalTime pattern ^\\s*Total time:), at a one-line summary, at a new Total tests line or at an invocation boundary, and pass any other line over without closing it. Add a TestCounters row with one foreign line between Total tests: 2 and Passed: 2 that wants discovered 2, passed 2."
  - id: F12
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/tierrun.go#filteredNoTestNotice
    summary: In a narrowed run over several projects, a project with no match prints that no test of the tier matched the filter even when another project of the tier matched it
    fix: "In filteredNoTestNotice, say that no test of the project matched the filter rather than of the tier: write no test of %s matched with project in place of the tier. Update the expected text of the two filtered rows in TestRunTierNoTestNotice to match."
  - id: F13
    severity: NIT
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/tiers.go#takeCallerFilters
    summary: A trailing --filter with no value stays in the args and renders beside {filter} as a second --filter, which dotnet test refuses
    fix: "In planTierRuns, beside the doubled --filter refusal, refuse a request whose last word after -- is exactly --filter, with the text: the words after \"--\" end in --filter with no expression; way forward: give the expression after --filter, then re-run the same command. Add the exact row to TestPlanTierRuns."
---

The diff builds every goal point of the bundled board entry (#216, #215, #235, #234, #233 and #237), and its commits carry `Root-Cause:` trailers that name the defects the fixes remove.
The findings below cover a constraint break in the new key names, untested parts of the new behaviour, a stencil sentence that reaches too far, and several small correctness gaps.

### [BLOCKING] New refusals spell the tiers key path as a literal

**Location:** `internal/gatecli/keynames.go#tierKeyName`, with the sites at `internal/gatecli/tiers.go:95`, `internal/gateslot/tiers.go:388` and `internal/gateslot/tiers.go:415`.

**Issue:** `PATTERN-config-key-names` requires a key name in a message to be rendered by the key-name helper from named constants, and `PATTERN-code-conventions` requires a name that can go stale to be one named constant.
The diff adds three refusals that build the path as `keyName("tiers."+…)`.
Older sites in `gateslot` use the same form, but the rule outranks that precedent, and the diff's own sites have to follow it.

### [MEDIUM] TestWithTimeoutMin does not pin the wall-clock term

**Location:** `internal/gatecli/tierrun.go#withTimeoutMin`, tested in `internal/gatecli/tierrun_test.go#TestWithTimeoutMin`.

**Issue:** The fake readings come from `time.Date` and carry no monotonic reading, so `at.Sub(start)` and `at.Round(0).Sub(start.Round(0))` are always equal in the test.
A mutant that keeps only `at.Sub(start)` passes every row, although counting wall-clock time is what lets a bound end after a host sleep.

### [MEDIUM] Planning's otherTiers is not asserted

**Location:** `internal/gatecli/tiers.go#withOtherTiers`.

**Issue:** `TestRunTierNoTestNotice` injects `otherTiers` directly, and no `TestPlanTierRuns` row reads the field.
If either return path of `planTierRuns` drops `withOtherTiers`, every test still passes, and in production the notice never names the tiers a filtered call did not run, which is #233's fix.

### [MEDIUM] The fixer's no-gate step covers every refusal

**Location:** `contracts/stencils/burler/burler-template-fixer.md:45`.

**Issue:** The paragraph opens "When `lym gate test` refuses to run any tier".
Every planning refusal is such a refusal, including those whose way forward is the fixer's own corrected call, such as the `--results-directory` refusal ("drop --results-directory from the words after \"--\", then re-run").
A fixer that reads the paragraph as written skips its gate and defers a step it could take itself.
#237 asks for a step when no gate exists, meaning a hub whose config gives the round no tier to run.

### [LOW] The copies prebuild keeps a monotonic bound

**Location:** `internal/gatecli/copies.go:66`.

**Issue:** `runTierCopies` still bounds the shared prebuild with `context.WithTimeout` over the tier's `timeout_min`.
The changed doc says that bound covers the prebuild and counts wall-clock time, which is false on this path.
Three lenses raised it; this one finding covers all three.

### [LOW] The Config.Tier and Config.Gate refusal texts are unasserted

**Location:** `internal/gateslot/config_test.go:883`.

**Issue:** The rows match only "the hub declares no tier0", so the key the diff adds to both messages is unpinned.
The mutation report's survivor at `gateslot/tiers.go:415` shows this.

### [LOW] observeFailure's new close at Total tests is untested

**Location:** `internal/gatecli/counters_test.go#TestCounters`.

**Issue:** No row runs a failure block straight into a `Total tests:` summary, so removing the clause the diff adds to `observeFailure` passes.

### [LOW] Budget path with a caller filter is untested

**Location:** `internal/gatecli/tiers_unix_test.go#TestRenderTierCommand`.

**Issue:** The budget row has no caller filter.
A mutant that falls back to the tier's filter alone past the budget drops the caller's expression and still passes.

### [LOW] TestWithTimeoutMin needs a keep marker

**Location:** `internal/gatecli/tierrun_test.go#TestWithTimeoutMin`.

**Issue:** The redundancy report lists it as contained by `TestRunBuildTimeout`.
`TestRunBuildTimeout` checks no poll length and no cause, so the test pins what coverage cannot see and is kept with a marker.

### [LOW] Refusal lists miss the doubled --filter refusal

**Location:** `internal/gatecli/tiers.go#planTierRuns`.

**Issue:** The doc comment and the package doc's list of planning refusals each enumerate the refusals, and neither names the new one.

### [LOW] An interleaved line ends a Total tests summary early

**Location:** `internal/gatecli/counters.go#observeSummary`.

**Issue:** The summary closes at the first line that is no outcome line.
`dotnet test` over several projects writes their output to one stream, so a foreign line can fall between `Total tests:` and its outcome lines, and that summary's counts are then lost.

### [LOW] The filtered notice speaks for the whole tier from one project

**Location:** `internal/gatecli/tierrun.go#filteredNoTestNotice`.

**Issue:** The notice is printed per invocation, but its text says no test of the tier matched the filter.
In a narrowed run over several projects, one project can match while another prints that line.

### [NIT] A trailing --filter is left in the args

**Location:** `internal/gatecli/tiers.go#takeCallerFilters`.

**Issue:** `--filter` as the last word has no value to take, so it stays in `{args}` beside `{filter}`, and `holdsCallerFilter` still counts it as a filter.

## Verdicts on the focus directive

- **`withTimeoutMin` and its readers.** Not a defect.
  `prebuildLym`, `tierTestTargets` and `exec.CommandContext` read only `Err` and `Done`, and no callee reads `Deadline` or `DeadlineExceeded`.
  A parent cancel is told apart by `ctx.Err() == nil`, a normal end leaves the cause `context.Canceled`, and the goroutine leaves on `Done`.
  The race between a non-zero exit and the bound passing behaves as it did with `context.WithTimeout`.
  The copies prebuild is F5.
- **The hung tier against #215 and #216.** Not a defect in the diff.
  On an awake host the old monotonic timer would have fired.
  `cmd.Cancel` kills the group, `WaitDelay` (10 s) bounds the pipe the orphaned test host holds, and in the memory-cap scope path every process left in the scope is killed after the SIGKILL (`proc/memorycap_linux.go`).
  I found no path that blocks past the bound while the host is awake, so the report that the bound did not fire before the host slept cannot be traced in code, and the Root-Cause trailer's two mechanisms account for everything I could reach.
  After the bound, `killMarkedLeftovers` ends the test host once `Wait` returns.
- **The marker kill after every tied child.** Not a defect.
  It reaches the build and every tier invocation, the two callers of `runTiedChild`.
  The marker holds this process's pid and a counter, so another gate's processes never carry it, and a reused pid would have a different environment.
  Gate runs set `proc.BuildServerOffEnv` (`run.go` `withBuildServerOff`), so no MSBuild reuse node or MSBuild server that another gate shares is meant to outlive a tier.
  A nil `cmd.Env` is replaced by `os.Environ()`, and an existing one is cloned before the marker is appended.
  Windows returns `ErrUnsupported`, which the job object covers.
- **The caller `--filter` merge.** F8, F10 and F13; the other `run.args` readers, among them green-record keys, `resultsArgsRefusal`, `emptyTierFailure` and the watch skip, read Go flags or the whole word list and stay correct.
- **observeSummary.** F7 and F11; no double count against the one-line summary, since a block and a one-line summary come from different logger verbosities and the results-file replacement resets the open block.
- **The no-test notice and otherTiers.** F3 and F12; both return paths fill the field, and `notRunTiers` leaves out guard and vet runs and covered tiers.
- **The help text cut.** Not a defect.
  The dropped clauses, the inert-only scope and the copies lease, are still stated in `internal/gatecli/doc.go`, and the flag help of `--copies` still names the one slot lease.
- **The stencil and the tier0 refusals.** F4 and F1; the refusal text reads correctly for every caller of `Config.Tier` and `Config.Gate`, each of which wraps it.

## Rejected

- The single-pass marker scan (code-correctness finding 3): only a process forked in the instant between the `/proc` read and the kill escapes. The doc's bound already admits that the kill is best effort, and a repeated scan would add cost to every tied child for no reported case.
- An indented `Total tests:` echoed from a test's stdout (part of code-correctness finding 1): the detailed logger prints test stdout under a "Standard Output Messages" heading, and a test that prints that exact line alone is no case the board entry or a report raises.
