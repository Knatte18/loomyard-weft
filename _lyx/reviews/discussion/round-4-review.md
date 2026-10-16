---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:84
    summary: D5 says a send landed with no turn start seen leaves the pause reading the session idle with only a missing resume line, but a turn start arriving after the wait lets the revived turn run uninterrupted under the live entry, and the Acceptance criterion on line 175 contradicts that case
    fix: In D5, replace lines 84-85 so the two unrecorded cases get separate consequences. For a send that lands with no turn start seen within the wait, state that the pause takes its entry when the wait ends, its grace poll and interrupt pass read the session idle and record nothing, and a turn start that arrives after that interrupt pass runs its turn under the live entry, uninterrupted and unrecorded, until the turn ends by itself; state the bound that `revive` reports such a session as sent with no turn start and `lyx tender status` shows it busy under the live entry, so the operator can pause that slug again. Keep the existing sentence for the turn that ends before the pause's poll. In Acceptance criteria line 175, change the first clause to read that a pause added while a `revive` line is in flight takes its entry only after that session's turn start reads or its turn-start wait ends, and interrupts and records the session when its turn start read within the wait.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:89
    summary: D5's bound says revive sends no key, but the guarded send it extends sends keys before and after typing - one interrupt-sequence key to close an open rewind picker, the clear sequence for a lone stray character in the box, and the clear sequence for its own unlanded text - and the conditioned send's at-once refusals name none of them
    fix: In D5's conditioned-send bullet (after the sentence listing the holds it refuses at once, line 75), add that the conditioned send also refuses at once, sending no key, when its ready reading shows the rewind picker or a lone stray character in the input box, and that it never clears its own unlanded text but reports the send as not landed with the text left in the box. Then the bound on line 89 that it sends no key holds as written. If the writer instead keeps today's picker close and clears, rewrite line 89 to name exactly those three keys as the only keys `revive` can send, and drop the claim on line 80 that a conditioned send only adds refusals.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:86
    summary: D5's lock-hold bound of two submit windows counts only from the start of typing, so it leaves out the ready reading before typing and the unlanded-text clear after the window, and every pane capture inside the windows runs under reed's operation lock and a tmux call that no window cuts, so the longest wait of a blocked pauserecord.Update is not what D5 states
    fix: Rewrite D5 line 86 to state the hold per send as one ready reading before typing (a fixed number of pane captures and event-file reads, with no settle when the send refuses a picker or stray character at once per F2, and no wait for a draft, operator activity or an unmatched turn start, each refused at that one reading), then at most one submit window from the start of typing to the landing, then at most one submit window for the turn start; and add that each pane capture inside that span waits on reed's per-worktree operation lock and a tmux call with no timeout of their own, so the stated windows bound the polling, not a hung reed or tmux call. Change line 87's at-most claim for a concurrent pause, resume or orch watcher write to refer to that bound.
  - id: F4
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:82
    summary: D5 has revive hold the pause record's write lock and change no record, but pauserecord's only locked entry point, Update, rewrites the file and prunes expired resumed entries, and D5 does not say how the lock is held without a write
    fix: In D5, after the sentence on line 82 that names the write lock, add one sentence choosing the mechanism - either pauserecord gains a call that takes the record's write lock, runs a function and releases it without writing, keeping its import set, or revive runs inside Update with a mutate that changes nothing and accepts its prune of expired resumed entries - and if the second is chosen, change change no record on line 89 to change no live entry.
---

### [MEDIUM] A turn starting after the turn-start wait runs under a live pause, unstated

**Location:** `_lyx/discussion/decision-record.md:84`

**Issue:** The pause's grace loop (`internal/pauseengine/pause.go`, the loop after `pauserecord.Update`) breaks at once when `anyBusy` reads every covered session idle, and `interruptSession` reads the same hook-derived `Busy`, which is `SessionState.RunningTurn()`: a `turn_start` signal moves the fold to busy (`internal/shuttleengine/sessionstate.go`, `applySignal`).
With every covered session idle-stalled and revive holding the lock, the pause's `Update` blocks until revive releases it.
When the send landed and no turn start read within the wait, the lock is released, the pause adds its entry, reads the session idle, lists it as already idle and leaves the interrupt step.
A turn start arriving after that point moves the session to busy under a live entry that never re-reads it, so the revived turn runs uninterrupted until it ends.
D5 lines 84-85 say only that the pause "reads either session idle and records nothing, so `resume` gives it no line", which is the right consequence for a turn that already ended, but not for a turn that has yet to start.
This is a guard the new verb lets a turn through, so under the rubric's attack-surface test its Decision must state what it lets through and what bounds it.
The Acceptance criterion on line 175 says a pause added during a revive line takes its entry "only after that session's turn start reads", which is false in this same case: the entry is taken when the wait ends.
Focus verdict: yes, with every covered session idle-stalled the Decision still lets a revived turn run under a live pause entry unrecorded, and the stated bound and criterion are not achievable as written for the late-turn-start case.
The turn-start-seen case is achievable: the same events file feeds both revive's wait and the pause's `Busy`, so a turn start revive saw reads busy at the pause's poll.

### [MEDIUM] Revive's "sends no key" bound is false for the guarded send it extends

**Location:** `_lyx/discussion/decision-record.md:89`

**Issue:** The conditioned send is today's guarded send plus refusals and a wait (line 80).
Today's send already plays keys outside the typed text: `closePickerBeforeSend` sends the first key of the engine's interrupt sequence when the rewind picker is on screen (`internal/shuttleengine/interruptguard.go`, `closeRewindPicker`), `clearStrayCharacter` plays the clear sequence on a lone stray character in the box (`internal/shuttleengine/send.go`), and `clearUnlandedText` plays the clear sequence on the send's own unlanded text after the window closes.
D5 lists the holds the conditioned send refuses at once as a draft, operator activity and an unmatched turn start; the picker and the stray character are neither, so as written revive still sends those keys.
Line 89's bound "it sends no key" is therefore not true of the design, and an operator who left a session's rewind picker open gets it closed by `revive`.

### [MEDIUM] The lock-hold bound omits the reading before typing and the uncut reed calls

**Location:** `_lyx/discussion/decision-record.md:86`

**Issue:** `pauserecord.Update` blocks on `state.UpdateJSON`'s file lock with no timeout (`internal/pauserecord/record.go`), so the longest time revive holds that lock is the longest time a `pause`, `resume` or orch watcher write waits.
D5 states it as two submit windows, one from the start of typing.
Before typing, the send's ready reading runs (`awaitIdleSession`): a pane capture, a possible picker close that sleeps the engine's interrupt settle and captures again, and a possible stray-character clear that sleeps `SubmitSettle` and takes a second full reading; none of that is inside either window.
After the first window closes unlanded, `clearUnlandedText` reads and clears the box, also outside the windows.
Inside the windows, every `CapturePane` runs under reed's `withOpLock` and a tmux subprocess (`internal/reedengine/io.go`), and `windowClosed` is only checked between calls, so a call blocked on reed's lock or a hung tmux extends the hold past both windows.
Focus verdict: for a draft, operator activity or an unmatched turn start on an idle-stalled `api-error` session the send refuses at its one ready reading, so the hold is that reading alone, and the Decision does not state even that; the stated maximum per send is not the longest hold.

### [LOW] How revive holds the record lock without a write is undecided

**Location:** `_lyx/discussion/decision-record.md:82`

**Issue:** `pauserecord` exposes the write lock only through `Update`, which always rewrites the file and drops resumed entries older than `Retention` (`withoutExpired`).
D5 has revive hold that lock across a send and also says it "changes no record" (line 89).
Either `pauserecord` gains a lock-only call or revive's hold rewrites and prunes the record; the Decision picks neither, and `PATTERN-leaf-packages` constrains what `pauserecord` may add.

## Rejected

- D6, focus on a caller context whose `WithTimeout` deadline passes during the hold first: not a defect.
  A `WithTimeout` context's `Err` becomes `DeadlineExceeded` at its deadline and stays so; a `cancel` called afterwards changes nothing observable, so "a later cancellation is not observable and only the moved deadline ends the call" (lines 100-101) is exact.
  A cancellation that came before the deadline leaves `Err` as `Canceled` even if the deadline also passes during the hold, so the transport can tell the two apart at release, and the claim "ends with the cancellation at release" holds for it.
  The Acceptance criterion on line 167 names only cancellation before the original deadline, matching the Decision; the deadline-first case is covered by the criterion on line 165.
- D8, focus on the watcher's memory-only handled set for an entry live at the re-exec with grace ahead: not a defect.
  `handleHubEntry` (`internal/orchengine/pause.go`) keys `pauseHandled` and `pauseFirstEsc` by live hub entries only, and `queueResumeLine` acts on resumed entries from the record's own `ResumeNoticed` flags, not from memory.
  D8 re-execs only at a tick where no hub entry is live, so no entry the old image handled is live at the exec; an entry added between that tick's read and the exec was never seen by the old image, holds nothing in its memory, and the new image handles it afresh, interrupting only before its grace end as line 126 says.
  A slug entry never covers the orch, per the file's own header, so "no hub pause entry is live" is the right condition.
- The support log's `## Review rounds` still reads `_No rounds yet._` after three rounds; the writer stencil assigns those entries to the Discussion-review gate, not to the writer, so it is not a defect of the target.
