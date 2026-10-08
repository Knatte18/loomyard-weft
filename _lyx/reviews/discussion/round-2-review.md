---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: "_lyx/discussion/decision-record.md D1, D3, D6"
    summary: "A send that fails with ErrSubmissionNotLanded leaves its typed text in the input box, and D1 refuses to type unless the box is empty, so D6's re-prompt retry hits ErrSessionBusy on every later tick and can never re-send; no Decision says the failed send clears its own text"
  - id: F2
    severity: MEDIUM
    class: design
    location: "_lyx/discussion/decision-record.md D6 (Decision and Bound), acceptance criteria"
    summary: "D6 clears the gate boundary on any turn_start after the turn end with no equivalent of D1's interrupt or 10 s idle override, so an interrupted writer turn with no Stop line stalls the loop to the deadline; the Bound claims the writer's next turn end becomes the next boundary, and the D6 acceptance criterion contradicts D1's override criterion"
  - id: F3
    severity: MEDIUM
    class: design
    location: "_lyx/discussion/decision-record.md D6 last Decision paragraph, D2, D3"
    summary: "D6 bounds the deadline overshoot by a fixed appearance check that no Decision defines or places relative to D3's submit window; the existing check plus its replay can run about 10 s, and D2 and D3 do not say whether the replay re-types inside the window or whether a read after the last Enter is guaranteed"
  - id: F4
    severity: MEDIUM
    class: decision
    location: "_lyx/discussion/decision-record.md D7 (handle mark, concurrent Wait), acceptance criteria on burler"
    summary: "D7 leaves open how burler reaches the handle form of the stop verb: burler holds only a Handle interface and stops a half by guid while that half's Wait is still running, so the in-memory handle is unmarked and its finalize writes died; Run also has no lock for the mark to sit under, and the burler acceptance criterion covers only the start-failure path where no Wait exists"
  - id: F5
    severity: MEDIUM
    class: design
    location: "_lyx/discussion/decision-record.md Constraints (PATTERN-completion-signal), D7"
    summary: "The Constraint adds the stop verb's file to the tripwire's scanned files but names no marker that makes the stopped write visible to the negative-verdict scan, and it leaves PATTERN-completion-signal's list of negative answers and wait.go's doc comment unchanged although stopped is a new negative answer"
  - id: F6
    severity: LOW
    class: consistency
    location: "_lyx/discussion/decision-record.md Constraints (PATTERN-completion-signal) versus Scope and D7"
    summary: "The Constraint says attach's classification is unchanged, but Scope lists attach's classification of a stopped record and D7 makes isTerminalOutcome recognise stopped"
  - id: F7
    severity: LOW
    class: consistency
    location: "_lyx/discussion/decision-record.md D1"
    summary: "D1 attributes TurnStartInterrupt to the engine that reads SessionSignalParser, but it is declared on SessionProber, so an engine with only the parser has no interrupt override; D1 also does not say whether api_error_turn_end counts as a turn end"
  - id: F8
    severity: LOW
    class: consistency
    location: "_lyx/discussion/decision-record.md D8"
    summary: "The PATTERN-shuttle-stop text says a shuttle run's strand is stopped only through the stop verb, but shuttleengine's own finalize cleanup, abandonStartup and start-failure teardown also remove strands; only the second clause scopes the rule to other packages"
---

### [BLOCKING] A failed submission leaves its text in the box, so D6's retry cannot re-send

**Location:** `decision-record.md` D1 (idle means the input box is present and empty), D3 (`ErrSubmissionNotLanded`), D6 (retry on later ticks), acceptance criterion "A gate re-prompt that does not land spends no attempt and is re-sent on a later tick".
**Issue:** `ErrSubmissionNotLanded` is returned when the box still holds the sent text after the window (D3), or when the box never settled and no Enter went out (D2).
In both cases the typed text is still in the box, and no Decision removes it.
D6 then retries on later ticks, and each retry starts with D1's idle wait, whose pane condition is an input box that is present and empty.
The retry therefore reads the leftover draft as busy and fails with `ErrSessionBusy` until the deadline, which is the unconsumed-paste case that Open risks names as a candidate root cause of #442.
The re-send only happens if the prompt is consumed late on its own, and then the writer's turn start clears the boundary anyway.
The same leftover blocks a follow-up `lyx shuttle send` to that pane.
**Fix:** Decide what a failed send does with its text.
Either the send clears the box when it fails, bounded to a box whose content is still the sent text, or D1's idle reading accepts a box holding exactly the previous failed text and replaces it.
State what the clear can delete, since it is a new key into a pane, and add an acceptance criterion for a retry after a not-landed re-prompt.

### [MEDIUM] D6 has no override for an unmatched `turn_start`

**Location:** `decision-record.md` D6 first Decision sentence and Bound; acceptance criteria for D1 and D6.
**Issue:** D1 releases a hold on a `turn_start` with no turn end after `TurnStartInterrupt` or 10 s of idle pane.
D6 uses the same signal condition at the boundary, but its action is to clear the boundary and keep polling, with no send.
After a writer turn that was interrupted, or whose `Stop` line is missing, no later turn end arrives, so no new boundary forms and the run idles until the deadline.
D6's Bound says a cleared boundary is progress because the writer's turn end becomes the next boundary, which does not hold for that case.
Before this change the first `Stop` evaluated the gate.
The acceptance criteria also disagree: the D1 criterion lets a send proceed once the turn is interrupted or the pane has read idle for 10 s, while the D6 criterion says no gate send is typed until a turn end follows the writer's newest `turn_start`.
**Fix:** Say whether D1's override applies at the gate boundary.
If it does, state that a boundary cleared by an unmatched `turn_start` is restored under the same two conditions, and align the D6 criterion.
If it does not, name the stall to the deadline in Open risks and fix the Bound.

### [MEDIUM] The deadline overshoot bound rests on an undefined appearance check

**Location:** `decision-record.md` D6 last Decision paragraph; D2 and D3.
**Issue:** D6 says in-loop sends overshoot the deadline "by at most its fixed appearance check".
No Decision defines that check.
In `run.go`, `sendVerified` runs `sendVerifyAttempts` polls of `sendVerifyInterval`, and re-plays the whole choreography `sendReplays` times, which is about 10 s with the current constants and types the text twice.
D2 and D3 start the submit window at typing and do not say whether the appearance check and the replay sit inside it.
They also do not say that a read after the last Enter always happens, so a settle that uses most of the window can send one Enter and fail without ever reading the result.
D6 caps the idle wait and the submit window "each" at the time left, which taken literally allows twice the time left.
**Fix:** Name the appearance check and its replay in D2 and D3, state whether they are inside the submit window, and state that the cap is taken once from the send's start, so the bound in D6 is the sum of what is left uncovered.
Add a guaranteed post-Enter confirm read, or state that its absence is accepted.

### [MEDIUM] How burler reaches the handle form of the stop verb is left open

**Location:** `decision-record.md` D7 ("Stopped through a run handle ...", "A `Wait` in another process ..."); the burler acceptance criterion.
**Issue:** `burlerengine` holds a `Handle` interface with `StrandGUID`, `RunDir` and `Wait` only, and `join` stops one half by guid while that half's `Wait` goroutine is still running.
D7 gives the in-memory mark only to the handle form, so a by-guid stop in the same process leaves the live `*Run` unmarked, and its `Wait` finalizes `died` over the `stopped` record.
D7 covers the other-process `Wait` case but not this in-process one.
The record also names a "handle's own lock", yet `shuttleengine` declares no mutex or atomic outside tests, so the lock is itself a new piece of design.
The burler acceptance criterion only covers the case where the second half's start fails, where no `Wait` exists, so it passes under either choice.
**Fix:** Decide that burler's stop goes through the handle form and say its `Handle` seam gains the stop, or decide that a by-guid stop also reaches an in-process handle.
Name where the lock lives, and extend the burler criterion to a half stopped during `join`.

### [MEDIUM] The completion-signal tripwire is named but not made to see the `stopped` write

**Location:** `decision-record.md` Constraints, `PATTERN-completion-signal` bullet; D7.
**Issue:** The Constraint says the stop verb's file joins the scanned files and its "return and call sites" are added to the pinned sets.
The negative-verdict scan pins return statements that contain one of its marker identifiers, and a record write that sets `stopped` returns none of them, so as written the scan sees only the `Errorf` returns in the new file and never the write that is the new negative answer.
A second writer of `stopped` would pass the tripwire unseen.
`PATTERN-completion-signal` and `wait.go`'s file doc comment list the negative answers as `OutcomeDied`, `OutcomeTimeout`, a mechanism-failure error and `verdictRespawnEligible`, and the Constraint does not add `stopped` to them, although the docs-in-the-same-commit rule covers it.
**Fix:** Say which identifier for `stopped` is added to the negative-verdict markers (or that the scan is extended to the record write), which function's `allOutputFilesExist` call goes into the call-site set, and that the pattern's background file and the `wait.go` doc comment list `stopped`.

### [LOW] The Constraint contradicts Scope on attach's classification

**Location:** `decision-record.md` Constraints, `PATTERN-completion-signal` bullet ("attach's classification is unchanged").
**Issue:** Scope lists "Attach's classification of a record lyx stopped" and D7 has `isTerminalOutcome` recognise `stopped`, which is what makes attach read it as respawn-eligible at any age.
**Fix:** Reword the Constraint to say the file-contract-first branch is unchanged and `isTerminalOutcome` is the only attach change.

### [LOW] D1 misplaces `TurnStartInterrupt` and leaves the turn-end kinds open

**Location:** `decision-record.md` D1.
**Issue:** `TurnStartInterrupt` is declared on `SessionProber` in `sessionsignal.go`, not on `SessionSignalParser`.
D1 names only the parser as the capability, so an engine with the parser and no prober has only the 10 s path, which D1 does not say.
`sessionsignal.go` also declares `api_error_turn_end` as its own kind, and D1's "newest turn end" does not say whether it counts, so a turn that ended on an API error could hold the next send for the override.
**Fix:** Name the prober capability in D1 and list the signal kinds that count as a turn end.

### [LOW] D8's text does not scope itself to code outside shuttle

**Location:** `decision-record.md` D8.
**Issue:** The invariant reads "only through shuttle's stop verb", but `finalize`'s done cleanup, `abandonStartup` and the start-failure teardown in `shuttleengine` remove strands without it, and the Out list does not mention them.
Only the final clause is scoped to other packages.
**Fix:** Word the first clause to cover callers outside `internal/shuttleengine`, or name the internal removers as the stop verb's own teardown.

## Focus verdicts that raised no finding

- D4 after `ErrSubmissionNotLanded`: D4 re-reads the box and the signals, then names the residual of a prompt consumed between that read and the strand removal, with the re-step's `Spec.validate` refusal as the consequence. That is a stated bound and is not a defect.
- D2 against D3: both windows start at typing, and a box that never settles returns `ErrSubmissionNotLanded` in both Decisions. The remaining gap is in F3.
- D6's per-tick trigger: the mark for an unsent re-prompt re-enters `handleGatedBoundary` through the existing `gatePending && gateAtBoundary` branch in `wait.go`, so it fires on a tick without a writer event.
- D7 and `allOutputFilesExist`: the Decision states that the verb consults it first, and the `running` record with its files present is still harvested by `dispositionCandidate`.
