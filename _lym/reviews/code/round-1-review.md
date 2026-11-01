---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/shuttleengine/claudeengine/background.go:345
    summary: OutstandingForks drops a fork launch whose tool result is not yet written, so the wait can answer ready true early
    fix: In OutstandingForks, count a fork launch with no recorded tool result (no ids yet) as outstanding, for example by having unfinishedForks scan launches itself or by returning pending launches from the shared scanner under a flag the Stop-payload fallback does not set. Add a TestOutstandingForks row with a fork tool_use line and no tool_result line that expects the fork outstanding.
    origin: lens:code-correctness
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/shuttlecli/awaitforks.go:110
    summary: The await-forks verb has no test for its envelopes, its no-strand and resolve errors, its argument rejection or its registration
    fix: In internal/shuttlecli/cli_test.go add a table-driven test for await-forks covering LYM_STRAND_NAME unset (error envelope, exit 1), an unresolvable strand (error envelope), ready true and ready false with outstanding (ok envelope), and an await-forks row with a positional argument in TestRunCLI_PositionalArgValidation. Drive it through a fake reed and runner the way the sibling shuttlecli tests do.
    origin: lens:code-tests
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/shuttleengine/await.go:266
    summary: Runner.AwaitForks has no test for its told-path error, unknown guid or engine without the OutstandingForkReader capability
    fix: Add a test in internal/shuttleengine/await_test.go that builds a Runner with a stub engine lacking OutstandingForkReader and asserts the capability error, one with an unknown guid asserting the not-a-shuttle-strand error, and one with a capable stub engine asserting the reader receives the run's SessionID and paneCwd.
    origin: lens:code-tests
  - id: F4
    severity: LOW
    class: design
    location: internal/shuttleengine/await.go:289
    summary: A fork that never gets a completion notification keeps the wait at ready false forever, and the stencils say to re-run it without an exit
    fix: In the three stencils (burler-cluster-fan, burler-fix-self-check, twill-directive), add that after a bounded number of consecutive ready false answers the agent ends its turn and takes the forks as they arrive, the same fallback the stencils already give for a failing verb. Keep the number in the stencil text only.
    origin: lens:code-correctness
  - id: F5
    severity: LOW
    class: design
    location: internal/shuttleengine/claudeengine/background.go:267
    summary: The changed turn-end reading for a session that never calls await-forks is pinned only at the scanner, not at the Stop event or session state
    fix: Add a row to the claudeengine events test that parses a Stop payload with no background_tasks list over a transcript holding a fork launch without run_in_background and expects EventWaiting with that fork in Outstanding.
    origin: lens:code-tests
  - id: F6
    severity: NIT
    class: design
    location: internal/shuttleengine/await.go:271
    summary: Any FindRun error, including a read or decode failure, is reported as not a shuttle strand
    fix: Word the error so it names the lookup failure without asserting the cause, for example by wrapping the FindRun error with the text find the run of strand and the strand guid.
    origin: lens:code-correctness
---

### [MEDIUM] OutstandingForks can answer empty while a fork is still launching

**Location:** internal/shuttleengine/claudeengine/background.go:345
**Issue:** `OutstandingForks` reuses `transcriptHasUnmatchedLaunch`, which skips a launch with no recovered task id.
A launch whose tool result is not yet in the transcript has no id, so it is not counted, and `awaitForks` returns ready true at the first read.
The capability's own doc comment says an empty answer ends a wait.
The stencils run the verb in the message after the spawn, so the result lines are normally present, but the reader does not guarantee it and no test covers it.

### [MEDIUM] The await-forks verb is untested

**Location:** internal/shuttlecli/awaitforks.go:110
**Issue:** The sibling `await` verb has `TestAwaitVerb` and a positional-argument row; `await-forks` has neither.
The no-strand error, the resolve error, both ok envelopes, `cobra.NoArgs` and the registration in `cli.go` are all unpinned.
The verb is the one entry point the three stencils depend on.

### [MEDIUM] Runner.AwaitForks is untested

**Location:** internal/shuttleengine/await.go:266
**Issue:** Only the inner `awaitForks` poll is tested.
The wrapper's three early returns and its hand-off of `SessionID` and `paneCwd` to the reader have no test.

### [LOW] No exit for a wait that never turns ready

**Location:** internal/shuttleengine/await.go:289
**Issue:** A fork killed or lost across a resume never produces a notification, so every call answers ready false.
The stencils say to re-run it in a turn that does nothing else, with no bound.
The run's own deadline eventually ends the role, so the cost is bounded but silent.

### [LOW] Turn-end behaviour change not pinned at the event level

**Location:** internal/shuttleengine/claudeengine/background.go:267
**Issue:** `isBackgroundLaunch` now counts a bare fork launch, and the transcript fallback feeds `events.go:49` and `signals.go:154`, so a Stop payload with no `background_tasks` list now reads as waiting where it read as done.
That matches how the payload path already treats subagents, and no defect was found in it, but only the scanner has a row for it.

### [NIT] FindRun error misreported

**Location:** internal/shuttleengine/await.go:271
**Issue:** The wrapper text asserts the strand is not a shuttle strand for every `FindRun` failure.

## Rejected

- Lens code-tests item on `taskFields` having no test: a pure extraction with unchanged output, and the `state` verb's tests already read its envelope; not a coverage gap.
- Lens code-affected-set F1 (await-forks verb untested): the same defect as F2, merged there.
- Lens code-tests F-T4 (no row for a fork with no result line): the same defect as F1, folded into its fix.
- Lens code-correctness item that a non-fork `run_in_background` Agent launch is also awaited: acceptable, the wait is for every background subagent the session launched, which is what the Goal needs.

## Holistic judgement

The diff makes the change the Goal asks for: the three stencils wait inside one verb placed after the spawn and, for the handler, after its holistic work, forks stay parallel, and no earlier rule is dropped.
The `fork-wait` measure is unchanged: `where-turns-go.sh` counts a no-call turn after a spawn, and the wait now fills those turns with a call, so the measure moves in the intended direction without any edit to it.
`docs/overview.md` and the package docs state the verb accurately.
The range's commit carries no `Pattern-Departure` trailer, none is needed, and no PATTERN entry is declared changed.
`PATTERN-shuttle-provider-seam` holds, since Claude specifics stay under `claudeengine`, and `PATTERN-daemon-wakeups` is met by the one-second floor and the cap.
