---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D4
    summary: D4 does not say how sender identity resolves on Linux when a worktree's strands or pane processes cannot be read, and the default outcome (EXTERNAL) is the privileged one
    fix: In D4, after the sentence ending "since every live strand's pane process is known", add "When the worktree list, a reed state or a strand's pane process cannot be read, a peer in no known strand pane is refused, naming what could not be read; it is never resolved as EXTERNAL or as a program sender." Add to the Linux identity Acceptance criterion "a peer in no known pane while a strand's pane process cannot be read is refused, not EXTERNAL".
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D9, D12, D15
    summary: post --wait-reply and inject --wait have no bound, and several paths never answer them
    fix: In D12, give `post --wait-reply` and `inject --wait` the `--timeout` that `read --wait` has (D8), with expiry exiting non-zero and printing that no outcome arrived; state that with no --timeout they wait until an outcome. In D15, after "it is marked undelivered and stays in the journal", add "and a `post --wait-reply` waiting on that message is answered that it returned undelivered, exiting non-zero." Add to the waiting-verb Acceptance criteria "a --wait-reply whose message returns undelivered is answered; --timeout on post --wait-reply and inject --wait expires non-zero".
  - id: F3
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md D9, D13
    summary: D9 does not say whether a hold stops the TTL of injects the hold mutes, so a hold of up to an hour silently expires every other sender's queued inject
    fix: In D9, extend "counted only while the receiver is live and no pause covers it" with the chosen rule for holds, either "and no other sender's hold covers it" (and add held and released to the transitions that append an aged event) or "a hold does not stop it, so an inject muted past its TTL by another sender's hold expires"; add the matching case to the inject TTL Acceptance criterion.
  - id: F4
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md D13, D16
    summary: Whether hold and release on an orch strand are accepted in stage 1 is unstated, though the hold exists for the watcher's orch cycle
    fix: In D16, after the sentence refusing inject and interrupt into an orch, state whether `hold` and `release` on an orch are accepted (recording the hold, with nothing typed) or refused naming `creel-notices`, and add that case to the orch Acceptance criterion.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D4 Bound
    summary: The off-Linux bound names only passing as EXTERNAL, but D4 also lets every off-Linux peer, one inside a strand pane included, claim a program-sender name
    fix: In D4's Bound, change "off Linux, identity is the environment alone, so an agent that unsets `LYX_STRAND_NAME` passes as EXTERNAL and gains D9's privilege" to "off Linux, identity is the environment alone, so an agent that unsets `LYX_STRAND_NAME` passes as EXTERNAL, and any process speaking the socket protocol may claim a program-sender name, each gaining D9's privilege". Make the matching Open risks line say the same.
  - id: F6
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D1
    summary: D1 sources strand liveness and pane process from persisted reed state, which records a pane id but no pane process or liveness
    fix: In D1, change "strands from each hub worktree's persisted reed state, read-only" to "strands from each hub worktree's persisted reed state, and their liveness and pane process from a read-only query of the hub's tmux server through reed", keeping tmux commands out of creelengine as D1 already states.
  - id: F7
    severity: NIT
    class: decision
    location: _lyx/discussion/decision-record.md D4
    summary: D4 does not say whether a program-sender claim must carry this hub's shortname
    fix: In D4, change "Any upper-case name but `EXTERNAL` is accepted." to "Any upper-case name but `EXTERNAL`, under this hub's shortname, is accepted; another shortname is refused."
  - id: F8
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md D6, D9, D10, D13, D15, D16
    summary: Outcome and event names drift (typed vs injected, held reused for orch-muted reminders), and holder names both a message's holder and a hold's owner
    fix: In D9 and D12 use the D6 event name `injected` for the typed outcome, or rename the D6 event to `typed`. In D16 call the orch's withheld reminders "withheld", not "held", since `held` is D13's hold event. In D10 and D13 call the sender holding a hold the "hold owner", keeping "holder" for a message's holder as D6, D11 and D15 use it. In D9, drop "or refused" from the `--wait` outcomes, or name the case that refuses a queued inject.
---

### [MEDIUM] Sender identity fails open when strand state cannot be read

**Location:** `_lyx/discussion/decision-record.md` D4

**Issue:** On Linux D4 resolves a peer by walking its ancestry against "the strands' pane processes" and refuses a pane-less `$LYX_STRAND_NAME` claim "since every live strand's pane process is known".
That premise fails when a worktree's reed state or its pane processes cannot be read.
D5 says a failed read fails the verb for receiver names, and D15 says a failed read moves nothing, but D4 says nothing.
A peer inside a strand pane whose worktree is unreadable then matches no pane and, by "Anything else is `<shortname>:EXTERNAL`", becomes EXTERNAL, which D9 makes privileged.
That is the in-pane escalation D4's rationale says the ancestry check closes, and the attack-surface rule requires a Decision to bound it.

### [MEDIUM] Waiting verbs with no bound and paths that never answer them

**Location:** `_lyx/discussion/decision-record.md` D9, D12, D15

**Issue:** This settles the round's fourth focus question.
`read --wait` has `--timeout` (D8); `post --wait-reply` and `inject --wait` have none.
- D15 sends a removed holder's unread message back toward its sender and, for an EXTERNAL sender, marks it undelivered, but nothing answers a `--wait-reply` waiting on that message, so it waits forever.
- A message read and never replied to also leaves `--wait-reply` waiting forever.
- `inject --wait` reaches an outcome for a removed receiver (D15 expires it), but its TTL runs only while the receiver is live and unpaused (D9), so a receiver that stays dormant in reed state, or one under a hub pause the operator never lifts, keeps `--wait` blocked forever.
  D9 names `inject --wait` as the confirmation `creel-notices` relies on, so a stage-2 caller would hang through a hub pause.
The reconnect cap in D12 bounds drops, not the wait.
The disconnected-waiter half of the question is settled: D12 answers a reconnecting waiter from the pool.

### [LOW] Inject TTL under another sender's hold

**Location:** `_lyx/discussion/decision-record.md` D9, D13

**Issue:** D9 counts an inject's TTL "only while the receiver is live and no pause covers it", and lists live, dormant, paused and unpaused as the transitions that append an aged event.
D13 mutes other senders' injects for a hold of up to an hour.
So by the record as written, the TTL keeps running under a hold, and a default 10-minute inject from another sender expires during the watcher's context cycle, the hold's stated use.
That may be intended, but D9's rationale names only pause and dormancy as holding the queue, and nothing says so.
The rest of the third focus question is settled: the aged event, the last-aged-event fold, D6's carried TTL and "time with the daemon down never counts" derive the consumed TTL after a restart, and an Acceptance criterion covers it.

### [LOW] Hold on an orch in stage 1

**Location:** `_lyx/discussion/decision-record.md` D13, D16

**Issue:** D13's rationale says holds exist for the watcher's context cycle, which holds an orch.
D16 refuses inject and interrupt into an orch and stores its mail, but says nothing of `hold` or `release` on one, and no criterion covers it.
The fifth focus question is otherwise settled: D10 refuses `interrupt` and `inject --interrupt` from a non-holder under a hold, naming the holder and expiry, and D13 refuses a second sender's hold the same way, each with a criterion.

### [LOW] Off-Linux bound omits in-pane program-sender claims

**Location:** `_lyx/discussion/decision-record.md` D4 Bound

**Issue:** D4 says "off Linux every peer may" claim a program-sender name, so an agent in a strand pane that speaks the socket protocol directly gains D9's privilege.
The Bound's off-Linux sentence names only unsetting `LYX_STRAND_NAME`, and the "every platform" sentence speaks only of processes outside a strand pane.
The second focus question is otherwise settled: on Linux, an orch pane's claim wins, any other pane's claim is refused, a pane-less claim of a strand's name is refused, and a pane-less program-sender claim is accepted, bounded by the OS user.

### [LOW] Pane process and liveness are not in persisted reed state

**Location:** `_lyx/discussion/decision-record.md` D1

**Issue:** D1's seam lists each strand's "live or dormant, pane process", and creelcli takes "strands from each hub worktree's persisted reed state, read-only".
The persisted `Strand` record in `internal/reedengine/state.go` carries `paneId` but no pane pid and no liveness; reed reads `#{pane_pid}` live from tmux (`internal/reedengine/overlay.go`, `paneListFormat`).
D4's whole Linux identity rule rests on the pane process, so the source D1 names cannot feed it.

### [NIT] Program-sender shortname unchecked

**Location:** `_lyx/discussion/decision-record.md` D4

**Issue:** D4 checks an off-Linux `$LYX_STRAND_NAME` against "this hub's shortname", but for a program sender says only "Any upper-case name but `EXTERNAL` is accepted", leaving a claim such as `other:BATTEN` open.

### [NIT] Event and role names drift

**Location:** `_lyx/discussion/decision-record.md` D6, D9, D10, D13, D15, D16

**Issue:** D6's event list has `injected`; D9 and D12 speak of a "typed" outcome.
D16 shows the orch's reminders "as held", while `held` is D13's hold event.
"Holder" is a message's current holder in D6, D11 and D15 and the hold's owner in D10 and D13, which makes "a receiver whose holder was removed" read two ways.
D9's `--wait` lists "refused" as an outcome, but every refusal the record names happens at request time, before the inject is queued.

## Rejected

- The first focus question, a retyped inject after a kill or a binary replacement: no defect.
  D9 journals and syncs a sending event before the send, and the fold treats a sending event with no outcome as unconfirmed, so a daemon killed after the keys were sent never retypes the inject, and the criterion "A daemon killed after an inject's send began…" catches it.
  On a replaced binary, D2 waits for in-flight sends and journals their outcomes before exiting; no criterion pins that wait, but the D9 fold rule rules out a retype either way.
