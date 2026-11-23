---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/shuttleengine/run.go:667
    summary: A resumed start whose attempt fails retryably sets both runs aside but retries with the same resumed spec, so every retry re-launches the session that just failed; the comment and summary claim the next attempt runs fresh, and no test pins this branch.
    fix: "In Runner.start, keep the fresh spec (spec with the carry-over block appended when carried is non-empty) and the pre-resume resumeWarning and transcriptCopied values beside the resumed ones. In the `if resuming && run != nil` branch, after the two setAside calls, switch spec, resumeWarning and transcriptCopied back to the fresh values and set resuming to false, so the retry starts a fresh session and the success path notices the carried instructions. Add a row to TestStart_ResumesAnInterruptedRunsSession (or a sibling test) whose first resumed attempt dies before its first input with the strand removed and StartRetries at least 1. Assert that the second Prepare call gets no ResumeSessionID, the prompt without interruptedResumePreamble and SkipSkillLoadTurn false, and that both the failed run's record and the predecessor's record carry NoResume."
  - id: F2
    severity: MEDIUM
    class: decision
    origin: lens:code-scope
    location: internal/shuttleengine/crashresume.go#resumeInterrupted
    summary: "The board entry asks for the resume \"for every role and engine\", but Codex runs keep starting fresh: codexengine.validateSpec refuses a ResumeSessionID and Codex has no SessionResumer, and the record holds no Added after Discussion entry narrowing the goal."
    fix: Leave the code in place and defer this finding to the run's parent. The parent either records the narrowing to engines with SessionResumer with `lym loom decision add`, or asks for a Codex resume path as follow-up work; both lie outside the fixer's write surface.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/attach.go:240
    summary: No test pins that a superseding attach records a running candidate's stop as Interrupted, which is what lets the start after it resume the session; the mutation survivor that negates the comparison passes every test.
    fix: In the attach tests that drive Runner.Attach with removeSuperseded over a respawn-eligible candidate (for example beside TestAttach_OutcomeDisposition), seed one candidate whose record reads running with a dead own-window pane. After the attach, assert that its record reads stopped with Interrupted true. Seed a second candidate with a terminal outcome such as died, and assert that its record keeps that outcome with Interrupted false.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: contracts/orchrole/orch-role.md:296
    summary: The orch role stencil still says a re-call after a resumed halt spawns a fresh agent, while after this diff a halt that followed a crash, a reboot or a killed pane resumes the row's recorded session.
    fix: "Rewrite the clause at contracts/orchrole/orch-role.md:296 so it says that a re-call after an escalation's answer spawns a fresh agent, that a resumed halt whose agent ended before its work did resumes that agent's session, and that lym sends the row-reached mail again in both cases."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/crashresume_test.go#TestStart_ResumesAnInterruptedRunsSession
    summary: No test pins that a resumed start keeps the carried operator instructions out of its prompt, still records them in run.json, and tells the notifier nothing.
    fix: Add a row to TestStart_ResumesAnInterruptedRunsSession whose resumable predecessor carries CarriedInstructions (for example a single literal instruction). Assert that the prepared prompt is exactly interruptedResumePreamble plus the task, that the new run's record lists the instruction under CarriedInstructions, and that the fixture's notifier received no carried-instructions notice.
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/stop_test.go#TestStop_RecordsTheStopBeforeRemovingTheStrand
    summary: No test pins that StopStrand records its stop with Interrupted false, the write that keeps a deliberate stop from being resumed.
    fix: In TestStop_RecordsTheStopBeforeRemovingTheStrand, after the existing outcome assertion on the stopped record, assert that record.Interrupted is false.
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/crashresume_test.go#TestRunner_RetireInterruptedRuns_SetsAsideOnlyResumableRuns
    summary: No test pins that RetireInterruptedRuns carries on past a failed write and returns the joined error, which the doc comment promises.
    fix: Extend TestRunner_RetireInterruptedRuns_SetsAsideOnlyResumableRuns, or add a sibling, with one resumable run whose record cannot be rewritten (made read-only through permkit.Chmod) beside a resumable one that can. Assert that the call returns a non-nil error naming the failed run directory, a count of 1, and NoResume on the writable run.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/loomcli/gotoresume_test.go:1
    summary: The file header claims the test pins the set-aside running before the PR-Review reset, but the test never observes the reset or the order.
    fix: Either add a seam that records the PR-Review reset call and assert that the set-aside call comes first and that the reset still runs after a failed set-aside, or rewrite the header to say the test pins that an admitted goto sets the runs aside once and that a failed set-aside leaves the goto standing.
  - id: F9
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/landing/summary.md:57
    summary: The Merge danger blast radius names only shuttle, but the diff also changes what an admitted `lym loom goto` and the arbiter's relaunch do in loomcli, and adds the interrupted and noResume keys to run.json.
    fix: "In the summary's ## Merge danger section, widen the blast radius to name shuttle's run start and run.json records and loom's goto and arbiter relaunch hook."
---

The diff puts the resume in shuttle's one start path, keyed by role key and output-file set, and distinguishes an interrupted stop from a deliberate one through `RunState.Interrupted` and `RunState.NoResume`.
It meets the board entry for the Claude engine: a died run, a waiterless running record and a step-stop all resume, while a goto, a new review round, a deliberate stop, a timeout and a missing transcript start fresh.
No finding is BLOCKING.

### [MEDIUM] A failed resumed start is retried as the same resume

**Location:** `internal/shuttleengine/run.go:667`

**Issue:** `start` swaps `spec` for the resumed spec once, before the retry loop (`run.go:644-649`).
When a resumed attempt dies before its first input and the teardown removed its strand, `startRetryable` holds (`run.go:687-688`).
The branch at `run.go:667-671` then sets both runs aside, and the loop launches the same resumed `spec` again with its `ResumeSessionID`, preamble and `SkipSkillLoadTurn`.
The comment on that branch and the change description both say the next start runs fresh instead of retrying the resume.
No test row drives a failed resumed startup, so neither the set-aside nor the retry behaviour is pinned; this merges the code-tests lens's missing-row finding into this one.

### [MEDIUM] Codex runs are left out of "every role and engine"

**Location:** `internal/shuttleengine/crashresume.go#resumeInterrupted`

**Issue:** The board entry's Idea asks for the resume "for every role and engine", on the premise that Codex takes a resume ID.
`codexengine.validateSpec` refuses one (`codexengine.go:103-104`), and the Codex engine has no `SessionResumer`, so `resumeInterrupted` always starts a Codex run fresh.
The change description names this as a writer's call ("provider work outside this change"), and the record carries no `Added after Discussion` entry narrowing the goal.
Whether to narrow the goal or build a Codex resume path is the parent's call.

### [MEDIUM] The superseding attach's interrupted stop is untested

**Location:** `internal/shuttleengine/attach.go:240`

**Issue:** The mutation report's survivor negates `c.state.Outcome == runOutcomeRunning` and every test passes.
That flag decides whether a run whose dead strand an attach removes stays resumable, the crash path where reed still tracks the dead pane.
Only `stop_test.go`'s step-stop assertions read `Interrupted`.

### [LOW] The orch role stencil promises a fresh agent after a resumed halt

**Location:** `contracts/orchrole/orch-role.md:296`

**Issue:** The stencil says "a re-call after an escalation's answer or a resumed halt spawns a fresh agent and sends it again".
After this diff a resumed halt whose row's agent died, lost its waiter or was step-stopped resumes that agent's session through `resumeInterrupted`.
The orch reads the line when it decides how to brief a row's agent.

### [LOW] The resumed start's carried-instruction handling is untested

**Location:** `internal/shuttleengine/crashresume_test.go#TestStart_ResumesAnInterruptedRunsSession`

**Issue:** `run.go:645-649` keeps the carry-over block out of a resumed prompt and `run.go:662` skips the notifier, while `launch` still records the instructions.
Every carry-over row in `carryover_test.go` seeds a stop not marked interrupted, and the crash-resume rows seed no instructions, so none of the three behaviours is pinned.

### [LOW] StopStrand's deliberate stop is untested

**Location:** `internal/shuttleengine/stop_test.go#TestStop_RecordsTheStopBeforeRemovingTheStrand`

**Issue:** `StopStrand` passing `false` at `stop.go:158` is what keeps a stop on purpose from being resumed.
The crash-resume row "a run stopped on purpose starts fresh" seeds its record by hand, and no stop test reads `Interrupted`.

### [LOW] RetireInterruptedRuns' failure path is untested

**Location:** `internal/shuttleengine/crashresume_test.go#TestRunner_RetireInterruptedRuns_SetsAsideOnlyResumableRuns`

**Issue:** The doc comment promises that a failed write does not stop the scan and that every failure comes back joined (`crashresume.go` lines 136-139 of the scan).
The test covers only a scan in which every write succeeds.

### [LOW] The goto test's header claims an order it does not pin

**Location:** `internal/loomcli/gotoresume_test.go:1`

**Issue:** The header says the test pins that the set-aside runs "before its PR-Review reset".
The test's receiver has a location, so the real reset runs, but nothing observes it or its order relative to the set-aside.

### [LOW] The change description's blast radius is narrower than the diff

**Location:** `_lym/landing/summary.md:57`

**Issue:** `## Merge danger` names the blast radius as shuttle alone.
The diff also rewires loom's goto and arbiter relaunch hook (`arm.go`, `arbiterconsume.go`, `gotoresume.go`) and adds two keys to run.json.
The two-way call itself holds: both keys are `omitempty`, records are tolerant, and an older binary only stops resuming.

## Rejected

- code-correctness finding 2 (a still-waited predecessor's `finalize` drops `NoResume`): the case needs a start of a row's output files while that row's earlier waiter is alive, and attach makes a running record with a live strand attachable while the loop's run lock keeps one step per row, so no second start takes that run's place.
- The `crashresume.go:92` mutation survivor: the negated condition only decides whether a warning is logged.
- The `run.go:651` mutation survivor: an awaited spec never shares the empty output set of an unawaited predecessor, so deleting the clause changes nothing observable.

## Focus notes

- Existing `ResumeSessionID` users: `resumeInterrupted` returns early on a caller-set `ResumeSessionID`, and the account-switch and re-prompt relaunches build their specs in `relaunchSpec` and `relaunchForReprompt` without passing through `start`'s new code, so their behaviour is unchanged.
- Stop changes: `StopStrand` passes `false`, `stopRecorded` writes the flag only on the running-to-stopped path, and `TestStopStrand_UnrecordableStopRemovesNothing` still pins that a stop it cannot record removes nothing.
- New review rounds start fresh through the output-file check, since each round names its own files; a re-call after an escalation's answer starts fresh because the escalated run ended done.
