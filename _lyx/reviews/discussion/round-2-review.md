---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:102
    summary: D7's SessionEnd path has no launch boundary, so an earlier launch's process-ending SessionEnd in the shared events file ends a resumed launch died before its own SessionStart(resume) lands
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:119
    summary: D8's residue reading applies only at the guard reading, and D8 leaves open how the post-paste confirm reads a box holding the residue character plus the sent text, which today fails the send as sent-text-in-draft
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:137
    summary: The PATTERN-pane-text-injection Constraints entry amends the background's never-wipes clause but not its lead sentence that lyx's text never merges with or submits a draft, which D8's residue path does
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/support-log.md:203
    summary: The Turn 8 ledger records D7's staleness fix as aging registry evidence against the newest SessionStart, the rule the Rejected alternatives now reject and D7 no longer states
  - id: F5
    severity: NIT
    class: consistency
    location: _lyx/discussion/support-log.md:243
    summary: The rejected treat-as-empty alternative is rejected because the exact-text check could not hold, which applies equally to D8's own residue branch
---

### [BLOCKING] D7's `SessionEnd` path fails a resumed launch on an earlier launch's line

**Location:** `_lyx/discussion/decision-record.md:102`, with the bound at line 111.

**Issue:** D7 gates registry death behind the wait having read the id alive, which settles the focus question on registry evidence: no units mismatch between `procStart` and signal times arises, a crashed launch's dead entry cannot fail a resumed launch, and the death criterion covers a live entry of the current launch reading dead.
The first bullet has no such gate: the run ends when the newest signal is a process-ending `SessionEnd` and the registry does not read the process alive, and line 111 says a wait attached after the crash ends on a `SessionEnd`.
reed relaunches a strand with its stored `ResumeCmd` (`internal/reedengine/strand.go`), which carries the same `--settings` file, so the resumed process appends to the same `events.jsonl`.
`claudeengine` reads `logout`, `prompt_input_exit` and `other` as process-ending (`internal/shuttleengine/claudeengine/signals.go`).
When the earlier launch ended with such a line, its registry entry is gone or dead, so `processLiveness` answers unproven or dead, never alive.
On the first liveness tick of a wait over the resumed launch, before the new process writes `SessionStart(resume)`, the newest signal is the old `SessionEnd`, and D7 ends the run died.
Line 110's claim that a resumed launch is never failed holds for the registry path only, and no Death criterion covers an earlier launch's `SessionEnd`.

**Fix:** Give the `SessionEnd` path a launch boundary the tick can test, for example by counting a process-ending `SessionEnd` only when this wait has seen a signal after it, or has read that session alive.
Restate line 110 to cover both paths, and add a Death criterion: an earlier launch's process-ending `SessionEnd` as the newest signal, with the resumed process not yet registered, ends nothing.
If D1 must record whether a pane kill writes `SessionEnd(other)`, name that in D1.

### [MEDIUM] D8 leaves open how the post-paste confirm reads residue plus the sent text

**Location:** `_lyx/discussion/decision-record.md:119`, and line 122.

**Issue:** D8 applies the residue reading at the send's guard reading.
After the paste, `confirmPasted` in `internal/shuttleengine/send.go` reads the box again: when the paste's Enter did not take and the character was real, the box holds `+<text>`, which fails the `box == normalized` match and fails the send with `ErrSentTextInDraft` through the `strings.Contains` branch, with no Enter.
The send is then left as a draft of lyx's own text behind the character, which holds every later send.
D8 neither extends the residue reading to that confirm read, which would put an Enter of lyx's own into a box not holding exactly the sent text, nor states that the send fails there and what clears it.

**Fix:** Decide in D8 which reading the post-paste confirm takes for a residue send, state the bound of whichever is chosen, and add the case to the Stray character criterion.

### [LOW] The Constraints entry does not amend the background's lead sentence

**Location:** `_lyx/discussion/decision-record.md:137` to `141`.

**Issue:** The background opens with "Text lyx types into an agent pane never merges with, submits or wipes an operator's draft".
The entry amends the never-wipes clause and the lone-`%` sentence, but D8's residue path merges a surviving real character with lyx's text and submits it, which the lead sentence still forbids.
The `%` sentence already contradicts the lead sentence today; D8 adds a second case without reconciling it.
The entry's claim that the `PATTERN.md` line is unchanged rests on the residue reading defining what the box holds, which also depends on F2's decision.

**Fix:** Name the lead sentence's merge and submit clauses among the amended places, bounded by D8's guard, operator-quiet and once-per-send conditions, and reconcile the `PATTERN.md` line wording with F2's outcome.

### [LOW] The Turn 8 ledger states a superseded D7 fix

**Location:** `_lyx/discussion/support-log.md:203`.

**Issue:** The line reads "fixed, registry evidence older than the newest `SessionStart` is stale", while the Rejected alternatives at lines 257 to 259 reject exactly that rule and D7 now requires an alive read first.

**Fix:** Correct the line to the alive-first rule D7 states.

### [NIT] The treat-as-empty rejection reason applies to D8 too

**Location:** `_lyx/discussion/support-log.md:243` to `244`.

**Issue:** The alternative is rejected because "the paste would submit it at the head of the message, and the Enter rule's exact-text check could not hold", which is what D8 does on its residue branch.

**Fix:** Reword the reason to what D8 adds over it: the clear makes the delivered text exact in every case but a character surviving the clear and the settle.

## Focus verdicts

- D7: the registry-evidence rule is sound in units and launch boundary; the defect is the `SessionEnd` path, F1.
- D8: the bound is stated in D8; the amendment record is incomplete, F2 and F3.
- D5: not a defect.
  D5 extends the interrupt release to an ask with cause `ask`, and a real transcript shows an escaped permission dialog writes `[Request interrupted by user for tool use]` as the newest main-chain entry, after the rejected `tool_result`, so `TurnStartInterrupt`'s existing marker test reaches it.
  D5's bound states the hold and both ways out, and the permissions Open risk matches it.
