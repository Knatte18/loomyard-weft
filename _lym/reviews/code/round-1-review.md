---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: handler
    location: internal/shuttleengine/loadturn.go:129
    summary: "A load turn still open at loadTurnOpenBound (15 minutes), or one with no turn start read, still ends as a skipped skill and types the prompt into a failing API"
    fix: "In loadSkillTurn, stop returning ended=false for a turn that watch.open shows as started and not ended: past loadTurnOpenBound keep polling until a turn end is read, the pane dies or the run deadline passes, and end a deadline expiry through run.classifyDeadlineExpiry(OutcomeTimeout) in a failed start, never by typing the prompt. A timeout skip stays only for a turn whose start was never read (an engine with no SessionSignalParser). Drop the 'skipped at the open bound' wording from doc.go, spec.go and the loadTurnOpenBound comment, and give the open-turn case a test row that runs past 15 minutes of virtual time with no turn end and asserts the prompt is not typed."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/startup_test.go:1691
    summary: "No row pins three behaviors of the hold, a turn start read during the back-off, the busy-session retype and the run and prompt offsets after a hold"
    fix: "Add rows to TestStartup_LoadTurnAPIError, not new tests. (1) A turn start written into the events file during the back-off: assert no second load line is typed and the prompt follows that turn's end. (2) A retype send that fails with ErrSessionBusy: assert the load is typed after the next, doubled back-off and the prompt still follows. (3) After a start held on an API error, assert run.offset and the persisted run state PromptOffset equal the events file size, so the re-typed load turn's start and end are never read by Wait as the prompt's."
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/startup_test.go:1771
    summary: "Remaining hold paths lack coverage and row 7 duplicates the existing hang row without pinning the open bound"
    fix: "Add to the same test: a row whose API errors run past the 5 minute ceiling asserting the wait stops doubling at loadHoldBackoffCeiling; a retype send failing with an error other than ErrSessionBusy asserting a died start; an interactive run and a notifier that returns an error asserting no notice and no failed start; and an endLoadHold assertion on the died and model-rejected returns. Remove row 7, which the existing hang row at startup_test.go:1594 already covers, or reshape it to assert the open bound as F1 asks."
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/shuttleengine/doc.go:321
    summary: "doc.go, the Run.deadline comment and the Runner notifier comments still describe the old timeout, deadline and notifier behavior"
    fix: "In doc.go replace the lines 321-322 claims (timeout measured from the end of the startup probe, a turn that never starts or ends is bounded by the timeout) with the behavior loadSkillTurn now has, and line 184 so the deadline also excludes a skill-load hold. In run.go update the Run.deadline field comment to name endLoadHold beside Wait as a mover of it, and the Runner.notifier and SetNotifier comments to say the notifier also receives the skill-load hold notice."
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: pattern/PATTERN-daemon-wakeups.md:25
    summary: "The daemon-wakeups entry describes only the back-off wake, though loadSkillTurn also wakes every pollInterval, and the new reed-panes section has no Rule or enforcement line"
    fix: "In PATTERN-daemon-wakeups.md extend the loadturn.go entry to say the loop also polls events and reed status every pollInterval between turns, with the floor that interval takes. In PATTERN-reed-panes.md give the new 'An API error holds the prompt' section a Rule line and an Enforcement line naming TestStartup_LoadTurnAPIError, as the 'A pause holds prompts' section does."
  - id: F6
    severity: LOW
    class: scope
    origin: handler
    location: internal/shuttleengine/loadturn.go:25
    summary: "The model-rejected end (loadModelRejectedError, abandonLoad, Result.APIError) and the parent notice line are not in the Goal's Wanted list"
    fix: "Record the two additions as parent corrections with lym loom decision add, or remove them. Do not revert them in the fix round: leave the code in place and defer this finding to the run's parent, who decides whether the model-rejected end and noticeLoadHold stay."
---

### [BLOCKING] A load turn still open at the open bound, or never seen to start, still types the prompt into a failing API

**Location:** internal/shuttleengine/loadturn.go:129
**Issue:** The Goal's second Wanted bullet is that the prompt is not typed into a session whose API is failing, and the incident was a skip at the timeout with a turn the API was failing. `loadSkillTurn` holds only once an API-error turn end has been read. A turn that is open when `timeout` passes is extended only to `loadTurnOpenBound`, 15 minutes after the call's entry; after that `settleLoadTurn` logs the timeout skip and `loadSkillsThenPrompt` types the prompt. The provider retries a failing API inside the turn for minutes, so an outage longer than the bound, such as the 13:39 to 13:55 drop that started the task, reaches the same skip and the same typed prompt as before, only later. A turn with no turn start read, for an engine without `SessionSignalParser`, also still skips at the 2 minute timeout. Nothing in the Goal licenses a 15 minute cap, and the focus directive's central question is answered no for the open-turn path: it is skipped, not held. The decision keeps the skip for a skill that is not installed or missing, which an open turn is neither.

### [MEDIUM] Hold behaviors without a pinning row

**Location:** internal/shuttleengine/startup_test.go:1691
**Issue:** `TestStartup_LoadTurnAPIError` pins the retype, the doubling, the transcript marker, the retry that spends no retry and a dying pane. It has no row that fails if `turnStartRead` stops cancelling a pending retype, if the busy-session retype stops moving `retypeAt`, or if the run offset and the persisted prompt offset stop passing the re-typed load turn. Each is a behavior the diff adds, and the offset one is what keeps `Wait` from reading the re-typed turn as the prompt's.

### [LOW] Remaining hold paths and a duplicated row

**Location:** internal/shuttleengine/startup_test.go:1771
**Issue:** The `loadHoldBackoffCeiling` cap, a retype send error other than busy, the notice skipped for an interactive run and the notifier error path, and `endLoadHold` on the died and model-rejected returns have no row. Row 7 sets `wantFailed` and returns before any time check, so it pins neither the open bound nor the continuation, and its skip is the one the existing hang row at `startup_test.go:1594` covers.

### [LOW] Stale prose after the behavior change

**Location:** internal/shuttleengine/doc.go:321
**Issue:** `doc.go` still says the load timeout is measured from the end of the startup probe and that a turn that never starts or ends is bounded by the timeout, and its deadline sentence at line 184 omits the hold; the new paragraph appended below contradicts both. The `Run.deadline` field comment still says only Wait moves the deadline, and the `Runner.notifier` and `SetNotifier` comments still say the notifier receives a held turn end only, though `noticeLoadHold` now uses it.

### [LOW] Pattern entries incomplete

**Location:** pattern/PATTERN-daemon-wakeups.md:25
**Issue:** The new entry names only the back-off wake, but between turns `loadSkillTurn` also wakes every `pollInterval` and calls `reed.Status`. The new "An API error holds the prompt" section in `PATTERN-reed-panes.md` has no `Rule:` line and no enforcement line, which its sibling sections carry. The back-off start of 30 seconds in the entry does match `apiErrorRetryBackoff`.

### [LOW] Model-rejected end and parent notice go beyond the Wanted list

**Location:** internal/shuttleengine/loadturn.go:25
**Issue:** The Goal asks to wait until the API answers and run the load again. The diff also ends a start on a rejected model with `OutcomeAPIError` and `Result.APIError`, and sends the parent a notifier line at the second API error. Neither is in the Wanted list or the Decision. Ending on a rejected model is a defensible exit, since no wait clears it, but it is an unrecorded parent correction and is filed for the parent.

## Rejected

- `endLoadHold` over-extends the deadline by the final successful load turn's own run time: the deadline then errs on the long side by one short turn, and the Goal asks only that API waiting not count against it.
- `abandonLoad` returns the run unprompted when every output file already exists: `Spec.validate` refuses a pre-existing output file, so the branch cannot be reached by a start.
- A failure reported both as a `StopFailure` signal and a marked `Stop` event would be counted twice: no engine in the tree reports one failure both ways, and the lens did not confirm it.
- The `cmd/lym/daemonwakeups_test.go` list entry needs a test of its own: the scan test is the guard.
