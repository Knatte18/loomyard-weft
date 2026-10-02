---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/discussion/decision-record.md D3, D4, D5, D6
    summary: The prompt count, last-prompt time and pending notify are not on disk, so D5's three-notice bound fails after a driver restart and D6's notify has no way to reach the gate
  - id: F2
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md D2, D5
    summary: The record never says whether a failed delivery-prompt send counts toward the three-prompt cap and the one-minute throttle, and the closure cannot learn the send's result
  - id: F3
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md D2, D4
    summary: Finalize's fresh gate evaluation runs the closure's state-changing rows, so a deadline can open an undeliverable request or consume a reject that is never sent, and D4's lost-reject bound omits this path
  - id: F4
    severity: LOW
    location: _lyx/discussion/decision-record.md D3, D9
    summary: A fresh Discussion-Write start after a dead writer opens the next round, and nothing closes the prior round, which stays open without a verdict
  - id: F5
    severity: LOW
    location: _lyx/discussion/decision-record.md D6
    summary: notify and delivered are not specified against an expired request or one that already has a verdict
---

### [BLOCKING] Per-round delivery bookkeeping is not on disk

**Location:** `_lyx/discussion/decision-record.md` D3, D4, D5, D6.

**Issue:**
D5 re-sends the delivery prompt "a minute after the last prompt", "up to three prompts per round", and states the bound "the parent receives at most three notices per round from the gate, plus one per `notify` run".
That bound needs two facts per round: how many gate prompts have gone out, and when the last one went.
D6's `notify` needs a third: a pending notify request, written by a separate process (the verb runs in the prime or the task worktree, never inside the driver) and read by the gate.
D3 lists the round directory's contents as exactly the request, the delivery record, the verdict and the review file, so none of the three is on disk.
D4 has the closure decide "from the current round's directory", which means the closure cannot see any of them.

I checked what a restart does in the code.
`reconstructAndWait` (`internal/shuttleengine/attach.go`) builds a fresh `Run` and replays `events.jsonl` from offset 0, so the writer's last Done replays as an arrival and the gate list runs again.
If the count and the last-prompt time live in memory, as `gateFails`/`gateSent` do today, the replayed arrival sees zero prompts sent and no last-prompt time.
The closure therefore carries the prompt at once, and up to three more follow.
Each restart adds three notices, so D5's stated bound holds only within one driver process.
`notify` has no mechanism at all: the record says it "has the gate send one delivery prompt" but never names the file it writes, who clears it, or whether two `notify` runs before one turn boundary yield one prompt or two (D5 says "one per `notify` run").

This is a stated bound that does not hold as specified, on a verb the attack-surface rule requires to be bounded.
A plan written from D3 alone would implement `notify` with no channel to the gate.

**Fix:**
Put the bookkeeping in D3's round directory and make D4's closure read and write it.
The delivery record gains the gate prompt count and the last-prompt time, both written by the gate when it carries a prompt.
`notify` writes a pending-notify count there, which the closure decrements when it carries a notify prompt.
Restate D5's bound as holding across driver restarts because the count is on disk, and add an acceptance criterion: a driver restart after two gate prompts sends at most one more.

### [MEDIUM] A failed send's effect on the cap and throttle is unspecified

**Location:** `_lyx/discussion/decision-record.md` D2, D5.

**Issue:**
D2 has the loop send a pending result's text, and on a failed send it logs a Warn and leaves the entry pending.
D5's cap and throttle count "prompts", but the closure that decides whether to carry one never learns whether the loop's send succeeded: `Gate` returns a `GateResult` and has no callback.
The two readings behave differently.
If the closure counts a prompt when it returns the text, three failed sends spend the cap with nothing delivered, and from then on only `notify` can reach the writer.
If only a successful send counts, the gate needs a way to report the send back, and without the throttle stamp a failing send is retried on every poll tick while the writer sits at the turn boundary, logging a Warn each time.

**Fix:**
Decide it in D5.
The simpler choice keeps the gate contract as it is: the closure stamps the count and last-prompt time when it carries the text, so a failed send counts toward the cap and the throttle.
The bound then reads "at most three delivery prompts attempted per round", and the failed-send Warn is what the operator sees, with `notify` as the way forward.
State the choice in D5 and add it to the existing failed-send acceptance criterion.

### [MEDIUM] Finalize can run the closure's state-changing rows

**Location:** `_lyx/discussion/decision-record.md` D2, D4.

**Issue:**
`finalize` is the gate's single verdict site, and it calls `evaluateGate`, which runs every closure afresh whenever the memo is nil (`internal/shuttleengine/wait.go`).
The memo is nil after a lost session and on a deadline path, and D2's poll-tick re-evaluation must clear it for each tick.
D2 says "`finalize` never waits", but D4's closure has three state-changing rows: opening a request, marking a reject consumed, and marking the request expired.
A finalize-time call can therefore:

- open a request that no delivery prompt will ever follow, which D9's status then reports as waiting;
- mark a reject consumed that no re-prompt will ever carry, because finalize never sends.

D4's "Bound on the consumed marker" covers only a dead driver and a failed re-prompt send.
A deadline that expires while a reject waits for the writer's mid-turn Done (D2) loses that review through neither of them, and Open risks does not cover it either.

**Fix:**
State in D2 or D4 that a finalize-time evaluation of `parent-review` opens no request and consumes no verdict, and that it reports the entry `waiting` from the round directory as it stands.
Leave the mechanism to the plan.
If the plan cannot make the closure read-only there, D4's bound must name the deadline path and Open risks' lost-reject item must include it.

### [LOW] A fresh start leaves the prior round open

**Location:** `_lyx/discussion/decision-record.md` D3, D9.

**Issue:**
D3 has a fresh Discussion-Write start open the next round number, and an attach continue the latest round.
A start also happens when the previous writer died and could not be attached.
Its round may be open with no verdict, and the parent may already have a fork reviewing it (D12).
Nothing marks that round closed, so D9's "a round has an open request without a verdict" stays true for it indefinitely.
Also, a parent verdict submitted for the old round lands on whichever request the verbs call "the open request".

**Fix:**
State in D3 that opening a new round marks any earlier round without a verdict as superseded, and that the verbs act on the latest round only.
D9 then reports only the latest round.

### [LOW] notify and delivered on a closed request

**Location:** `_lyx/discussion/decision-record.md` D6.

**Issue:**
D6 has `approve` and `reject` refuse on a request that already has a verdict or has expired.
For `notify` and `delivered` it only says "refuses when no request is open".
`notify` on an expired or verdict-recorded request queues a prompt the closure never carries, since those rows pass.

**Fix:**
Give `notify` the same two refusals as `approve` and `reject`, and let `delivered` refuse on an expired request.
The acceptance criterion listing the refusal rows already names "request expired" generically, so it needs no change.

## Focus departures

None.
The directive's four questions are settled as follows.
Where the count, last-prompt time and pending notify live: nowhere on disk (F1).
Whether D5's bound holds after a restart: no (F1).
How notify reaches the gate: unspecified (F1).
Whether a failed send counts toward the cap: unspecified (F2).
