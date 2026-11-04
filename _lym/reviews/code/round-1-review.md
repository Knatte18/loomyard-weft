---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    origin: lens:code-affected-set
    location: internal/websterengine/verifygate.go:420
    summary: "The webster verify gate never reads Result.RunnerCrash, so a repeat crash still goes to the webster verify fixers and spends an attempt, against the board entry and the new PATTERN-gate-fixer text."
    fix: "In internal/websterengine/verifygate.go, after the TimedOut check on each verify result (first run and rerun), test res.RunnerCrash and return a terminal failing GateResult (Passed false, Terminal true) whose findings name the crashed runs from res.Detail and the log path and say no fixer runs, without the internal second verify rerun and without building a VerifyGateReport for the fixer fork or cold fixer. In internal/webstercli/verify.go runVerify, when res.RunnerCrash is set, return an error envelope that names res.Detail and the way forward of running `lym webster verify` again, instead of the fix the failures message. Add rows for both beside the existing timeout rows. If webster is meant to stay out of scope, narrow the PATTERN-gate-fixer wording in PATTERN.md instead and name the narrowing in the commit message."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/run.go:150
    summary: "The runTest closing-line rule (RunnerCrashLine only when every failed run crashed and no failing run, lint or blocked build judged), exitCodeOf's new signal branch and detachRun.endedOnRunnerCrash have no test; every landing and loom site reads that line."
    fix: "Add rows to the existing runTest tests in internal/gatecli (or a table-driven test beside them) for a call whose only failed tier crashed twice (line printed), a crash beside a failing tier, a crash beside a lint finding and a crash beside a build-blocked tier (no line). Add a runWithFixer row in the detachfixer tests where stderr holds gateslot.RunnerCrashLine and the seam spawns no fixer. Add an exitCodeOf row for a child ended by SIGSEGV (139) and one by SIGKILL (1)."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/verifytree/verify.go:419
    summary: "Verify setting RunnerCrash and Detail from the log's runner crash line, with the tier timeout line taking precedence, has no test, though every landing and loom site depends on that reading."
    fix: "In the verifytree tests, add table rows that run Verify over a command whose log holds gateslot.RunnerCrashLine (expect StatusFailed, RunnerCrash true, Detail naming the runs), one holding both a tier timeout line and a crash line (expect TimedOut true, RunnerCrash false), and one holding neither (RunnerCrash false)."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/landingshed/fixerrun.go:291
    summary: "The Parent-Align crash halt and the merge fixer's turn-gate crash end (fixerEndCrash and its haltReason) have no test; only Finalize's halt and the check mapping are pinned."
    fix: "Add a Parent-Align test that scripts a gate result with RunnerCrash true and requires a halting Stuck with the LandingStopPrefix reason, no fixer session and no landing failure record. Add a fixer-session test where the gate after a turn reports a crash and require the fixerEndCrash end, a halt reason equal to the gate's reason and no re-prompt."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/loomshed/verifygate.go:141
    summary: "Terminal set from res.RunnerCrash, which is what keeps the row's gatefixer wrap from spawning a fixer, has no test; the loom test asserts only the findings text."
    fix: "In internal/loomshed/verifygate_test.go, add a row or test that drives NewVerifyGate with a verify result of RunnerCrash true and requires GateResult.Terminal true, and a plain failed result that requires Terminal false."
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/tierrun.go:219
    summary: "A shortage in the first run followed by a crash in the rerun is labelled a runner crash in its run and its rerun and stops with no fixer, though only the rerun crashed."
    fix: "In retryTier, set retried.crash and retried.record.RunnerCrash only when the first run was a crash (signal != empty), and add a TestRetryTier row for a shortage then a crash that expects no crash label."
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: internal/gatecli/run.go:675
    summary: "exitCodeOf is shared with the build step, so a build killed by a crash signal now exits 128 plus the signal instead of 1, a change the exit-code paragraph of doc.go does not describe."
    fix: "Either state in the Exit codes paragraph of internal/gatecli/doc.go that a build ended by a crash signal also reports 128 plus the signal, or keep the old code for the build by leaving the signal branch out of the helper the build calls."
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/landingshed/verifygate.go:312
    summary: "A repeat crash at the pr-review gate tells the agent to end its turn again and halts nothing, and the failure is mailed once per head, so a persistent crash is not stopped with the crash named as the board entry asks."
    fix: "Decide and state the pr-review behaviour on a repeat crash in the headFailureReason comment; if the board entry's stop applies, have gateHead in internal/landingshed/prreview.go halt the row with the crash named when checkHead reports a runner crash, instead of mailing the agent."
---

### [BLOCKING] Webster verify gate still hands a repeat crash to a fixer

**Location:** internal/websterengine/verifygate.go:420, internal/webstercli/verify.go:69
**Issue:** The diff teaches Finalize, Parent-Align, the merge fixer's turn gate, the loom row gate and the card fixer about a runner crash, and writes `PATTERN-gate-fixer` as "a runner crash gets none". The webster verify gate handles only pass, skip, dirty and timeout, reads no `RunnerCrash`, reruns a crashed verify itself and then fails with the crash parsed as ordinary failure output, so the verify fixer fork and the cold fixer spend an attempt on a green tree, the harm the board entry names. `lym webster verify` likewise reports a crash as a failure to fix. The board entry's third expectation, a crash that repeats stops the run with the crash named, not a fixer, is unmet on this path.

### [MEDIUM] Closing-line rule, exitCodeOf and endedOnRunnerCrash are untested

**Location:** internal/gatecli/run.go:150, internal/gatecli/run.go:675, internal/gatecli/detachfixer.go:125
**Issue:** `TestRetryTier` pins detection and the rerun, but the call-level rule that decides whether the crash line is written has no test, and a regression there silently sends a crash to a fixer or hides a real failure. The signal branch of `exitCodeOf` and the card fixer's skip on a crash line are also unpinned.

### [MEDIUM] Verify's reading of the crash line is untested

**Location:** internal/verifytree/verify.go:419
**Issue:** `runnerCrashDetail` and the precedence of the timeout line over the crash line have no test in the diff, and every landing and loom site depends on them.

### [MEDIUM] Parent-Align crash halt and merge fixer crash end are untested

**Location:** internal/landingshed/parentalign.go:198, internal/landingshed/fixerrun.go:291
**Issue:** Only Finalize's halt and `check`'s mapping are covered. The Parent-Align branch and `fixerEndCrash` with its `haltReason` case have no row that would fail if broken.

### [MEDIUM] Loom row's Terminal flag on a crash is untested

**Location:** internal/loomshed/verifygate.go:141
**Issue:** The no-fixer guarantee of the loom row rests on `Terminal: res.RunnerCrash`, and the test asserts only the findings text.

### [LOW] Shortage then crash is labelled a repeat crash

**Location:** internal/gatecli/tierrun.go:219
**Issue:** `retryTier` reads the rerun for a crash whatever the first run was. A shortage followed by a crash reaches `failedRunLabel` as a runner crash "in its run and its rerun" and the call closes with the crash line, though the first run was no crash.

### [LOW] Build exit code changes with the shared helper

**Location:** internal/gatecli/run.go:675
**Issue:** `build.go` calls the same `exitCodeOf`, so a build ended by a crash signal changes its exit code from 1 to 128 plus the signal, with no description in the Exit codes paragraph.

### [LOW] pr-review gate does not stop on a repeat crash

**Location:** internal/landingshed/verifygate.go:312
**Issue:** The reason asks the agent to end its turn again, which runs the gate again. The mail goes once per head, so a persistent crash neither halts nor is repeated to the agent, unlike the other sites.

## Rejected

- Merge fixer keeps the turn's `Spent` charge on a crash (lens:code-correctness F3): the fixer session did run and its attempt is real; the escalate refund exists because an escalate is no fix attempt. The board entry asks that a crash spend no fixer, not that a spent one be refunded.
- Terminal on a `pass_on_cap` verify entry is a gate-evaluation error (lens:code-correctness F4): a recipe row putting `pass_on_cap` on a verify gate is a misuse of the entry's own contract, which the trust-the-caller rule leaves undefended.
- `endedOnRunnerCrash` reads the whole stderr file unscoped (lens:code-affected-set F2): the lens found no defect, since the line is written only when no judged failure exists.
- Wording test for `headFailureReason` and the `WaitStart` and cancel-during-delay rows (lens:code-tests items 7 and 8): the first is static prose, and the rest are covered in shape by `TestRetryTier`'s record and rerun assertions; a separate row would repeat them.
- `tier_runs` counting a rerun tier twice (lens:code-affected-set): two runs happened, and the shortage rerun already behaved this way.
