---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D5 (owners of a dead own-window strand), D11, Acceptance criteria
    summary: D5 names shuttle's supersession, a step stop and a seat table's stop as the removers of a dead own-window strand, but all three remove a strand only when reed reports it live, so a dead own window is never removed outside a pair teardown
    fix: In D5, replace the premise with a decided remover. Decide that shuttle's stop path (the stop verb, the step stop, supersession on respawn, and so a seat table's stop) removes a tracked own-window strand whose pane is present but dead, not only a live one, and state that bound (it removes only the strand the run record names, never an untracked pane). Name what removes a dead own window in the prime, where no pair teardown runs, and what removes one after a run that ended timed out or died with no later stop. Add an acceptance line that a seat table whose advisor died still ends its shared window when the table returns, and that a step stop of a run whose agent died ends its own window.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D3 (invalid window_split fails remove)
    summary: D3 makes remove fail on an invalid window_split or negative window_share_max, so a reed.yaml typo blocks every strand removal in the worktree, including shuttle cleanup, a seat stop and D12's self-removal, leaving agents running
    fix: In D3, take remove out of the ops that fail on an invalid window_split or window_share_max. Decide that remove always kills the strand's pane and ends its window as D5 and D6 say, skips only the even-layout of a shared window it leaves, and reports the invalid key in its result. Amend the matching acceptance line to list up, add and resume only, plus a line that remove under an invalid window_split still removes the strand.
  - id: F3
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md D2 (live count and the group record's lifetime)
    summary: D2 rests on a group's live count and its recorded choice and size, but does not say whether a strand bound to a dead pane, which D5 now keeps bound, counts toward the live count, nor when the group's record is dropped
    fix: In D2, state that a group's live count counts its strands bound to a present pane, dead or alive, or only alive ones, and state that the group's recorded choice and WindowSize are dropped when the group's last strand is removed, so a later instance under the same group name records afresh.
  - id: F4
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D12 (Bound, prime lifetime)
    summary: D12's prime bound says an opened session lives until its provider exits, but when the chained self-removal fails the pane goes dead and stays under D5 with its strand bound, and Resume after a server restart relaunches it, since an opened strand has no done-when paths
    fix: In D12's Bound, state what happens when the chained self-removal fails - the dead pane and strand stay until lyx reed remove, and Resume relaunches the opened strand only when its pane was alive when the server died - or decide another rule, and add the failed self-removal case to the open verb's acceptance line.
---

### [BLOCKING] Dead own-window strands have no remover that reaches them

**Location:** `_lyx/discussion/decision-record.md` D5, D11 and the Acceptance criteria lines on dead own windows and seat tables.

**Issue:** D5 keeps a dead own-window pane bound "until the strand's owner removes it (shuttle's supersession on respawn, a step stop, the pair's teardown)", and states that "the table removes every advisor strand before its step returns, so a shared seat window ends with its table".
In the code those removals all go through a liveness check that skips a dead strand.
`internal/shuttleengine/stop.go` `removeStrandIfLive` returns nil without removing when `!strand.Live`, and the stop verb, `StopStrand` and `stopRecorded` all end in it.
`internal/shuttleengine/stepstop.go` `StopStepRuns` skips a run whose strand is not live.
`internal/shuttleengine/attach.go` `removeSupersededStrands` skips a candidate whose strand is not live.
`internal/seatengine/engine.go` `stopSeat` calls the handle's `Stop`, which reaches `removeStrandIfLive` as well.
`Live` in reed's `Status` (`internal/reedengine/lifecycle.go`) is `aliveIDs[s.PaneID]`, false for a dead-but-present pane, which is the state D5 keeps.
And `Wait` removes the strand only on `OutcomeDone` without `KeepPane` (`internal/shuttleengine/wait.go`), so a timed-out or died run leaves it.
Today reconcile's dead-pane kill clears those panes at the next op; D5 and D6 withdraw that kill from own windows without a replacement, so the pair teardown is the only remover that works.
In the prime, which has no teardown, every agent that dies or times out leaves a window for good, and Resume after a server restart relaunches its strand.
A seat table whose advisor died leaves its shared window up after the table returns, against the acceptance line that the window "ends when the table returns".

On the focus question itself: the record does decide that the dead-pane kill and the keep-one survival rule stay limited to window 0 once the listing is session-scoped (D5, D6), and that decision agrees with D5's client select, since the only window end left is the removal path that runs the select.
The defect is the premise about who removes the dead strand, not the window-0 limit.

### [MEDIUM] An invalid window_split blocks every removal

**Location:** `_lyx/discussion/decision-record.md` D3.

**Issue:** D3 decides that an invalid `window_split` or a negative `window_share_max` "fails every reed op that reads it (up, add, remove, resume)".
Removal is the teardown path of shuttle's cleanup, the stop verb, a seat table's stop and D12's chained self-removal.
`Wait`'s cleanup logs a failed `RemoveStrand` as non-fatal and moves on (`internal/shuttleengine/wait.go`), so a `reed.yaml` edit on a live session leaves every finishing agent running in its window, and a seat stop fails with `ErrSeatNotStopped`, whose way forward, `lyx reed remove`, fails the same way.
The only work remove needs the key for is the even-layout of a shared window after a leave; the kill and the window end do not depend on it.
On the focus question: the record does decide what add and remove do with an invalid `window_split` on an already-up session, and what a join past the recorded `WindowSize` or after a `window_share_max` edit does (D2); this finding is about the remove half of that decision.

### [LOW] A group's live count and its record's lifetime are undefined

**Location:** `_lyx/discussion/decision-record.md` D2.

**Issue:** D2 sends a joiner apart when it "would push a shared group's live count past its recorded `WindowSize`", and refuses a size mismatch against "a live group", with the choice and size recorded on the group by its first strand.
D5 now keeps a dead own-window pane bound to its strand, so whether such a strand counts toward the live count decides whether a re-stepped table after an advisor died shares its window or scatters.
The record also does not say when the group's recorded choice and size are dropped, and D11 names a group by table prefix and round, so a re-step of the same round reuses the name.

### [LOW] D12's prime bound assumes the self-removal succeeds

**Location:** `_lyx/discussion/decision-record.md` D12, Bound.

**Issue:** D12 bounds an opened session in the prime by "until its provider exits or the operator runs `lyx reed remove` on it".
The provider's exit ends the session only through the chained detached `lyx reed remove`.
When that removal fails, the shell exits, the pane goes dead under `remain-on-exit`, D5 keeps it bound and no reconcile kill reaches it, and `planResumeLaunches` (`internal/reedengine/lifecycle.go`) relaunches every non-alive strand whose done-when paths are not all present, which an opened strand never has.
The session the operator ended then comes back after a server restart.
The rest of the focus question holds: D12 states no cap, a lifetime bound for both the pair and the prime, and accepts Resume bringing a live opened session back.
