---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/startup_test.go:2257
    summary: Startup retry bounds and refusal after paste lack behavior coverage.
    fix: >-
      Extend TestStartup_SendFailuresTearDown with virtual-clock rows for exhausting the permitted busy retries,
      a deadline that expires during a retry, and a busy sentinel produced after the paste.
      Assert the bounded virtual duration, no prompt typed after expiry, and exactly one paste for the latter case.
      Make the exhaustion row distinguish the permitted retry count from both fewer and additional attempts.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/attach_test.go:1155
    summary: Lost-prompt tests omit fresh-start recovery and the new recovery guards.
    fix: >-
      Extend TestWait_RetypesAPromptLineThatStartedNoTurn through public Start/Wait as well as Attach.
      Add rows for a pause beginning after initial delivery, an engine without session-signal parsing,
      a different instruction starting a turn after delivery, and a failed retype followed by further ticks.
      Assert fresh-start recovery, no typing while covered or without a parser,
      suppression after the later turn start, and exactly one attempted retype on failure.
      Include an attach whose operator turn predates the dropped delivery to cover F5.
  - id: F3
    severity: BLOCKING
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/startup_test.go:2358
    summary: Failure-capture tests omit styled capture selection and capture-error fallback.
    fix: >-
      Extend the existing startup failure table with an engine implementing StyledBoxReader
      and distinct plain and styled captures, asserting the exact styled bytes saved on delivery failure.
      Add a failed failure-capture row asserting that the earlier startup capture remains the saved artifact.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/shuttleengine/run.go:894
    summary: Startup retries and lost-prompt recovery can type after the run deadline.
    fix: >-
      Give sendStartPrompt and the send initiated by retypeLostPrompt a context bounded by the run deadline.
      Check expiry before initiating these sends, and retain the bound when sendThroughPause rebuilds its context.
      Apply the existing pause-time accounting when refreshing that bound.
      Keep public unrestricted sends unchanged, and use the F1 and F2 behavior tables to assert the boundary.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-affected-set
    location: internal/shuttleengine/hold.go:201
    summary: An earlier operator turn disables recovery of a newly dropped attach prompt.
    fix: >-
      Record a separate transient event offset immediately before each prompt delivery being watched,
      alongside promptSentAt in the startup and pending-delivery paths under sendMu.
      Make retypeLostPrompt search for subsequent turn starts from this delivery offset.
      Preserve the durable PromptOffset and the prompt-specific check used to identify the task's own turn.
      Add the historical-operator-turn case to the existing lost-prompt behavior table as described in F2.
---

### [BLOCKING] Startup retry boundaries are not pinned

**Location:** internal/shuttleengine/startup_test.go:2257

**Issue:** The new successful-retry row becomes idle after one send window and checks one prompt typing.
The always-active row checks eventual teardown but does not distinguish the retry count or elapsed virtual duration.
The table leaves the run timeout unset.
Its foreign-box row supplies a nonempty box before the send, so it does not exercise a refusal after a paste.
Changing the retry cap or broadening the retry to the post-paste busy sentinels can escape these assertions.
These are uncovered behaviors under PATTERN-test-speed's test-economy rule, which the directive makes BLOCKING.

### [BLOCKING] Fresh-start recovery and recovery guards are not pinned

**Location:** internal/shuttleengine/attach_test.go:1155

**Issue:** Both new rows use Attach and an engine that parses session signals.
Deleting the timestamp assignment in run.go:878 would disable fresh-start recovery without changing either result.
The rows cover successful retyping, its single-retype cap, and suppression by the prompt's own turn start.
They do not cover a pause after delivery, a non-parsing engine, another instruction's later turn, or retry failure.
The existing attach pause test covers initial delivery, before there is a timed retype to suppress.
These missing behaviors violate PATTERN-test-speed's test-economy rule.

### [BLOCKING] Capture selection and fallback are not pinned

**Location:** internal/shuttleengine/startup_test.go:2358

**Issue:** The sole added capture assertion uses an engine without StyledBoxReader and a successful capture.
It cannot detect selecting plain bytes for a styled reader or discarding the boot capture on a capture error.
Those are new observable branches in run.go:908–913, not static prose or duplicate helper coverage.
Their missing coverage violates PATTERN-test-speed's test-economy rule.

### [MEDIUM] New sends can outlive the run deadline

**Location:** internal/shuttleengine/run.go:894

**Issue:** Run.newSendContext in send.go:92 leaves deadline zero.
The new retry loop checks run.deadline only after its send returns.
With 65 seconds remaining and a 60-second idle window, the second attempt starts at 60 seconds.
If the pane becomes idle at 70 seconds, that attempt can type the prompt and return success after expiry.
If it stays busy, the attempt can consume its full second window.
The newly added retype has the same omission through run.go:989.
Wait calls it before checking expiry at wait.go:455, so it can begin another send when the deadline is already reached.
The send helpers have deadline clamps, but these contexts never activate them.

### [MEDIUM] A historical turn is mistaken for a response to the latest delivery

**Location:** internal/shuttleengine/hold.go:201

**Issue:** Pending delivery intentionally accepts a record with an operator turn after its durable PromptOffset.
The operator-turn rows in run_test.go:566 pin that behavior, and burlerengine.ResumeFix also calls this API.
After attach types a pending prompt, typePendingPrompt preserves that nonzero historical offset.
If the provider drops this new prompt, the watch finds the older operator turn and clears promptSentAt permanently.
That turn happened before the watched send and cannot establish that the new submission landed.
The successful-delivery latch also prevents another ordinary delivery through the handle.
This leaves the restarted strand without its task despite the new recovery path.

## Focus departures

- Directive: "Scope against the board entry: the entry's first part asks for bounded polling or retry of the idle check; does the diff cover the idle-detection mismatch on the saved `startup-capture.txt` layout, or only the retry?"
  The two named historical run directories are absent in this worktree, so their exact layout could not be checked.
  I read Claude's input-box parser and its idle fixtures instead.
  The diff adds retries and future failure captures; it does not change the parser.
  No available capture establishes a layout defect, so I raise no speculative parser or scope finding.

## Validation

The reviewed range is e723347087ca68e87b937be3136d2faca303b384..HEAD.
All four lens reports were saved before consolidation.
The pending-delivery split retains its locking and successful-delivery latch.
The late-landing exception already has a turn start, so omitting its new timestamp does not lose an eligible watch.
The retry's typed busy-error selection does not retry the post-paste busy sentinels in the current implementation.

The focused tier0 gate was attempted for the changed startup and lost-prompt behavior tests.
It could not acquire a slot because the hub's _bolt/.lym/gate directory is read-only in this sandbox.
No test executed in that attempt; the findings above are grounded in source and assertion review.
