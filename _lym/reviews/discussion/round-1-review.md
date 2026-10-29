---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md Open risks bullet 1, D1; _lym/discussion/support-log.md Question ledger Q10
    summary: The mode check D1 rests on is left for the tier3 smoke run although the installed source settles it now as CLAUDE_CODE_SIMPLE or --bare, which no lym session can carry
    fix: In D1's rationale add a bullet stating that the Stop runner skips the continuation push only when Claude Code's simple mode is on (the CLAUDE_CODE_SIMPLE env var or the --bare flag; in that mode the attachment is also dropped before the API call), that the same guard stands in the installed 2.1.310, 2.1.311 and 2.1.312 builds, and that lym never passes --bare and reed's Claude launch scrubs every CLAUDE_CODE_-prefixed variable (internal/shuttleengine/claudeengine/scrub.go, applied in internal/reedengine/lifecycle.go), so no lym session reaches it. Delete Open risks bullet 1, and mark Q10 resolved (research) with that answer.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D1 rationale bullet 3 (operator reads it once)
    summary: With systemMessage dropped the operator's only Stop copy is the stop-hook summary row, which Claude Code 2.1.312 clamps to 5 sub-lines, so a longer claim shows the operator a cut copy and the Goal's operator-once is unmet for it
    fix: In D1 either state the bound (the operator's copy is the stop-hook summary's Stop hook feedback lines, clamped to 5 lines by Claude Code 2.1.312's renderer, with the full text in the transcript view) and judge it acceptable against the Goal, or pick an operator copy that shows the whole claim and record its rendering evidence; add the chosen behaviour for a claim over 5 lines to the D1 acceptance criterion.
  - id: F3
    severity: LOW
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md Open risks bullet 2
    summary: The version risk's bound, a smoke rerun on a Claude Code upgrade, names no trigger although Claude Code updates itself without lym noticing, and the evidence that the Stop guard is stable across the installed versions is not recorded
    fix: Rewrite Open risks bullet 2 to record that the Stop continuation guard and the asyncRewake exit-2 rule are the same in the installed 2.1.310, 2.1.311 and 2.1.312, to state that Claude Code updates itself and nothing in lym reruns the smoke on an update, and to name the failure a breaking update would cause (mail journaled read in a turn that ends, visible only as D5's acknowledged line with no continued turn); keep the rerun as an operator action stated as such, not as a bound.
  - id: F4
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D3, D4 and their acceptance criteria
    summary: The record never states what the operator sees for a ring wake, so the Goal's operator-once is not shown on the ring path, and no criterion checks it
    fix: In D4's rationale state, from Claude Code 2.1.312's source, what the terminal draws for an asyncRewake wake (the rewakeSummary line, and whether the stderr body is shown under it), and add to the D3 real-session acceptance criterion that the operator sees the mail once on a ring wake.
  - id: F5
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D3 (reserved until the client acknowledges); D5
    summary: The daemon's hook-wait answer path can return on a closed connection or shutdown while the ring outcome sits unread in the waiter's channel, and D3 does not say that this exit releases the claim, so the mail could stay reserved until the daemon restarts
    fix: Add to D3 that every way the ring's answer path ends without a confirmed acknowledgement releases the claim, including the client leaving or the daemon stopping after the ring was answered but before the outcome was written, and add that release to D5's released outcome.
  - id: F6
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D3 second Bound; Constraints bullet each message is still delivered once
    summary: The daemon journals the ring's mail read before it writes the confirmation, so a lost confirmation makes hook-wait exit 0 and Claude Code drop the stderr, losing mail as read in a window D3's bounds do not name
    fix: Extend D3's second Bound to name this window too (the daemon journals the read, then its confirmation does not reach hook-wait, which exits 0, so the mail is read and never shown), with D5's acknowledged line as its record, and adjust the Constraints exception sentence to name both loss windows.
  - id: F7
    severity: NIT
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D3, The ring is answered only by that acknowledgement
    summary: Existing paths also mark a ring answered, a probe reading a turn in progress and any boundary claim that hands out mail, so only is false
    fix: Reword to say the ring's own claim marks the ring answered at the acknowledgement, not at hand-out.
---

### [MEDIUM] Mode-check research deferred to the smoke run

**Location:** `_lym/discussion/decision-record.md` Open risks bullet 1 and D1; `_lym/discussion/support-log.md` Q10.

**Issue:** The record leaves the mode check that gates Stop additional context for the tier3 smoke run.
That is a deferred research question in the rubric's sense, and research settles it now.
In the 2.1.312 Stop runner, `if(!Ugt(xe))He.push(xe)` guards the push into the continuation list.
`Ugt` returns true only when `Sr()` holds, and the runner's module imports `Sr` from the chunk where `Sr()` is `CLAUDE_CODE_SIMPLE || --bare`.
The same guard stands in 2.1.310 and 2.1.311.
reed scrubs `CLAUDE_CODE_`-prefixed variables from the Claude launch, and lym passes no `--bare`.

### [MEDIUM] Operator's only Stop copy is clamped to 5 lines

**Location:** `_lym/discussion/decision-record.md` D1 rationale bullet 3.

**Issue:** The `stop_hook_summary` renderer builds the `Stop hook feedback:` lines and passes them through `Is(..., {maxLines:eu})` with `eu=5`.
D1 removes `systemMessage`, the one full-text operator copy today.
So a claim of more than 5 lines shows the operator a truncated copy, and the record calls D1's copy "once" without that bound.

### [LOW] Version-upgrade bound has no trigger

**Location:** `_lym/discussion/decision-record.md` Open risks bullet 2.

**Issue:** Three Claude Code versions sit side by side in the installed versions directory, so updates land without lym's involvement.
lym has no version check, so "run again on a Claude Code upgrade" does not bound anything.
The cross-version evidence that would narrow the risk is not recorded.

### [LOW] Ring wake's operator copy unstated

**Location:** `_lym/discussion/decision-record.md` D3, D4 and their acceptance criteria.

**Issue:** The asyncRewake handler enqueues `Bin({summary, body, priority:"next"})` as a task-notification.
D4 names only the summary label, and nothing says whether the operator sees the mail body once, twice or not at all.
D1 and D2 each state the operator's copy, but D3 and D4 do not.

### [LOW] Ring answer path's release on early exit unstated

**Location:** `_lym/discussion/decision-record.md` D3 and D5.

**Issue:** `answerLocked` puts the outcome into the waiter's buffered channel and drops the waiter.
`awaitOutcome` can then take `watcher.closed` or `d.done` and return without writing or releasing (`internal/creelengine/server.go` `awaitOutcome`).
Today the outcome carries nothing, but under D3 it carries a reservation that only `Reading.Ack` or `Reading.Release` clears.
The acceptance criterion names the dropped-connection result, but the decision states no rule for this race.
The finding drops from the lens's MEDIUM to LOW because the criterion already pins the outcome.

### [LOW] Ack journaled before the confirmation

**Location:** `_lym/discussion/decision-record.md` D3 second Bound and Constraints.

**Issue:** `handOut` calls `reading.Ack()` before writing the confirmation, and `awaitAckConfirmation` returns `connLostError` when that write is lost.
D3 then exits 0, and Claude Code drops stderr on exit 0.
The mail is journaled read and never shown, in a window D3's bounds do not name.

### [NIT] "Only" overstates how a ring is answered

**Location:** `_lym/discussion/decision-record.md` D3.

**Issue:** `wakeHookStrand` calls `markRingAnswered` on a probe that reads a turn in progress, and `ClaimAtBoundary` sets `mark.answered` on any hand-out.
The ring's own claim is the only one D3 changes.

## Rejected

- interactions finding 2, that a ring to an unmarked waiter gets no D5 log line: that ring carries no mail, so it is no delivery attempt, and the bell that follows it is logged under D5. It occurs only while an older binary runs during a deploy.
- Focus item 3 (D1 count and cap under continuation): checked by the handler and every lens, and it holds. Claude Code pushes the context into `blockingErrors`, raises its own count against a cap of 8 and resets that count on a tool turn. Creel's Stop count still rises on each mail-handing Stop with `stop_hook_active` true, so the cap of 6 bounds a continuation loop as D1 states.
- Focus item 4 (relocation): no Decision or Scope text was judged deliberation narrative. D1's and D3's source bullets are the evidence the rubric asks a researched fact to carry.
