---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:53
    summary: D3 tests RequestedAt against the cycle's PhaseEnteredAt, but every phase entry overwrites it, so the test keeps requests the cycle covered and removes requests it did not
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:120
    summary: D7 records the batten-watched fact in the run's seed, but the seed is write-once and compared for equality against what the child's own bootstrap writes, which refuses a seed with an extra param
  - id: F3
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:122
    summary: D7 removes the driver's run-stopped message with no bound for a batten that is dead or whose notice was not queued, and batten marks a notice sent even when it was not queued
  - id: F4
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:128
    summary: D8 adds a second window to a reed session whose pane enumeration, reap and layout target the session's current window, and states neither detached creation nor how reed keeps addressing its strand window
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:131
    summary: D8 says the window closes when batten exits, but every reed session sets remain-on-exit on, so an exited batten leaves a dead window unless the op overrides it for that window
  - id: F6
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:80
    summary: D5 finds the stop report as the newest file in the drive-reports directory or the park marker path with no tie to the stop that changed state, so a notice can carry an earlier stop's report, and since is defined as entry time but read from the newest history entry
  - id: F7
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:79
    summary: D5 holds a state-changed notice for the report while D4 lets the done arm return Done as soon as the driver strand ends, and no Decision orders the pending notice before the return
  - id: F8
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:105
    summary: D6 states only the no-live-run fallback as what bounds quiet, while D4 names quiet as the bound of an unlimited wait; busy-but-not-progressing agents, a hung live wait, unreadable transcripts and several live runs are not covered
  - id: F9
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:101
    summary: D6 puts transcript-derived API error text into a notice typed into the orch session with no one-line or length bound, and a notice with a newline is refused and then marked sent
  - id: F10
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:66
    summary: D4 does not name the seam for batten's own pause flag, what a running child with a dead driver and an unchanged status file waits on, or how a failed stat differs from a failed read, and it runs the notice probes on every 2 s check
  - id: F11
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:145
    summary: D4 makes every Stuck budget-exempt and drops the 12-hour window, while the Constraints say bounce and exempt semantics are unchanged and the recipe row keeps or loses max_bounces unsaid
  - id: F12
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:49
    summary: D2's rationale says a restarted watcher needs a current turn end, but D1's completion uses only the held turn end's transcript path, so D2 is not needed by D1 or by any acceptance criterion
  - id: F13
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:124
    summary: D7 claims one channel per event, but an awaiting child still produces both the driver's parent_notice relay and batten's D5 state-changed notice
  - id: F14
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:157
    summary: A1 and A8 are not assumptions, since A1 is evidence already in D1's rationale and the support log and A8 restates D3's rationale
---

### [BLOCKING] D3 compares against a time that moves with every phase

**Location:** `_lyx/discussion/decision-record.md:53` (D3), `internal/orchengine/watcher.go:297` (`enter`).
**Issue:** D3 removes a pending request "whose `RequestedAt` is at or after the cycle's `PhaseEnteredAt`".
`enter` overwrites `PhaseEnteredAt` on every phase entry, and a compact cycle passes through handoff-requested, compacting and resuming.
At completion the value is the entry time of the last phase, or of the compacting phase if completion means the compaction landing.
A compact request made while the handoff was being written, or while `/compact` ran before it was typed, is older than that value and is kept, so the second compaction D3 exists to prevent still starts.
A compact request made during the reload after the boundary is newer and is removed, though the compaction it asked for was already over.
The Bound says "while that very cycle ran", which the stated test does not implement.
The same applies to a clear cycle, where the covering action is `/clear` typed in the clearing phase.
Clock source is not the problem: the request time (written by the CLI) and the phase time (the watcher's clock) are both wall clock on one host.
**Fix:** Name the cycle's start, persisted in `State` when the cycle enters handoff-requested, and the instant the same-mode action took effect: the boundary's time for compact, the confirmed `/clear` for clear.
A request is satisfied when it is made at or after the cycle's start and no later than that instant.
Remove it at that instant, and state the bound in those terms.

### [BLOCKING] D7's seed record contradicts the seed contract

**Location:** `_lyx/discussion/decision-record.md:120` (D7), `internal/shedrun/seed.go:187`, `internal/battencli/wire.go:340`.
**Issue:** D7 says "batten's Seed-Child records it" in the run's seed.
`WriteSeed` is idempotent only against a seed equal in recipe, driver and params, and the child's own bootstrap writes the seed again with only `params.parent`.
`childSeedParams` carries a comment saying exactly this: a child seeded with a param its own bootstrap does not write makes that bootstrap refuse.
A watched marker in `Params` therefore makes the driver's bootstrap fail with a disagreeing-seed refusal, and the seed is write-once so it cannot be added later either.
**Fix:** Record the fact outside `seed.json`: an ephemeral marker under the run's scratch directory (declared by `shedrun` beside the park marker, written by Seed-Child, holding batten's pid), read by the driver prompt render, which counts the run as watched only while that pid is live.
Keep "decided in Go at render time".
Name the marker and its liveness test in D7, and keep the seed unchanged.

### [BLOCKING] D7 can leave the orch with no notice and nothing bounds it

**Location:** `_lyx/discussion/decision-record.md:122` (D7 Bound), `internal/battenshed/notice.go:146`, `internal/battencli/wire.go:467`.
**Issue:** D7 removes the driver's "run stopped" message for a batten-watched run and bounds the removal with "only where a batten notice replaces it".
Three paths leave no replacement.
The driver's stencil is rendered once, so a batten that dies after that leaves a quiet driver and no notice.
`noticeStep` saves the episode key after `Notify` returns, also when it returned an error, so a failed notice is never retried.
The wired `Notify` discards the `queued` result of `QueueNotice`, which is false with no orch strand recorded, so an unqueued notice counts as sent.
Before D7 the driver's own message covered each of these; after it, nothing does.
**Fix:** State in D7 and D5 that a notice is recorded as sent only when `QueueNotice` reported it queued, so the next check retries it, bounded to one Warn per episode.
State that a batten started again sends the notices not yet sent, because the episode marker records only sent ones.
Add the remaining case to the Open risks: a batten that dies after its driver rendered leaves the orch silent until batten runs again.
The liveness test in F2's fix covers a batten already dead when the driver is rendered.

### [BLOCKING] D8's window and reed's single-window enumeration

**Location:** `_lyx/discussion/decision-record.md:128` (D8), `internal/reedengine/overlay.go:271`, `internal/reedengine/doc.go:246`.
**Issue:** reed's `listPanes` runs `list-panes -t =<session>:`, which names the session's current window, and doc.go states that every pane in a reed session is Selvage or a bound strand's pane, enforced by the untracked-pane reap.
A new window becomes the current window unless it is created detached, and an operator who selects it by hand makes it current again.
Reed would then enumerate the batten pane alone: the strands' panes look gone, and the batten pane is untracked and reapable.
D8 states neither `-d` nor any change to how reed addresses its strand window, and the board note `reed-multi-window` records that reed has no window support today and that a new window must not steal focus.
The Out list defers multi-window anchoring of strands, which does not cover this minimum.
**Fix:** In D8, create the window detached and never select it, and make reed's enumeration, layout and split ops address the strand window by its id rather than by the session's current window.
Add the criterion that a reed op run while the batten window is current leaves the strands untouched and the batten pane unreaped.
Keep strand anchoring in other windows out of scope.

### [MEDIUM] D8's window closing and duplicate bound

**Location:** `_lyx/discussion/decision-record.md:131` (D8), `internal/reedengine/lifecycle.go:480`.
**Issue:** reed sets `remain-on-exit on` globally for every session, so a window whose batten exited stays as a dead pane, contradicting "the window closes when batten exits", while the next sentence ("a window whose process has exited is replaced") assumes it does.
The Bound says the verb never starts a duplicate, but the window check is a read followed by a create, and two calls in the same moment pass it; the per-run lock that batten takes is the real bound.
`batten:<slug>` carries a colon, so it is a poor target for tmux's `session:window.pane` syntax.
**Fix:** Say the op sets `remain-on-exit off` for that window, so it closes on exit, and that a dead pane found under the name is replaced.
Name the run lock as what stops a second batten for one slug.
Say the name is for display and the op addresses the window by the id `new-window` returns.

### [MEDIUM] D5's stop report and since are not tied to the stop

**Location:** `_lyx/discussion/decision-record.md:80` (D5).
**Issue:** For a done child the report is "the newest file in the run's drive-reports directory".
That directory is durable and holds the reports of every earlier stop, so when the child turns done the newest file already exists and is an earlier stop's, and the notice goes out at once with a stale path.
The park marker has the same gap: nothing says the marker found belongs to this stop and not to an earlier park the driver has not yet removed.
`since` is defined as "when the child entered its current state" but read from the newest history entry, whose `last_at` is the last fold, not the entry into the state.
**Fix:** Bound the lookup: a report counts only when its file time (or the marker's) is at or after the state change's time, and `report none yet` is sent otherwise after the 3 minutes.
Define `since` as the newest history entry's time and say it is a lower bound on the state's age.

### [MEDIUM] D5's held notice and D4's Done return

**Location:** `_lyx/discussion/decision-record.md:79` (D5), `_lyx/discussion/decision-record.md:66` (D4).
**Issue:** D5 sends the done notice once the report exists, the driver strand has ended or 3 minutes passed, and D4's done arm returns `Done` once the driver strand ends.
Both conditions are the same moment, and after `Done` the engine tears down and batten exits.
No Decision orders the pending notice before that return, nor before any other return (a state change, a resume), so the done notice, and with D7 the only notice a done run produces, can be lost.
**Fix:** State that a call sends its pending notice before it returns for any reason, and that the done arm sends before returning `Done`.

### [MEDIUM] D6's bound names only one of the cases quiet now misses

**Location:** `_lyx/discussion/decision-record.md:105` (D6 Bound), `_lyx/discussion/decision-record.md:72` (D4 Bound).
**Issue:** D4 removes the time limit on a running child and names the quiet notice as one of its bounds, and A3 relies on quiet and pause for an unwatched run.
D6 replaces the status-file test, which flagged a run with no step progress, with an activity test that by its own acceptance criterion never fires while the transcript grows.
A run in a loop that writes to the transcript without finishing a step is no longer flagged by anything, and the 12-hour window that stopped it is gone.
A hung live wait marker is excluded from quiet and left to `verify-watchdog`, which is out of scope.
D6 also does not say whether a live run with no readable transcript counts as idle or active, how several live runs combine (one active hides a stalled one from quiet), or that a dead pid is skipped so an API error in a dead run is never reported.
Only "no live run found" has a stated fallback.
**Fix:** State each case in D6 as a stated limit with its disposition: unreadable transcript, several live runs (quiet needs all idle, api-error needs any), dead pid, and a run busy without progress or in a hung wait, which only the operator's `lyx batten status` and pause bound.
In D4's Bound, name those limits instead of claiming quiet bounds the wait, and restate A3 to match.

### [MEDIUM] D6's API error text enters the orch's input unbounded

**Location:** `_lyx/discussion/decision-record.md:101` (D6).
**Issue:** The `api-error` notice names "the error text" read from a transcript and queued to be typed into the orch session.
`QueueNotice` refuses a line with a newline or a leading `/`, and `noticeStep` marks a refused notice sent, so an error text with a newline loses the very notice D6 adds.
Text from a provider reaching an agent's input is also an attack surface the attack-surface question asks a Decision to bound.
**Fix:** State that the error text is cut to one line and a fixed length by the same rule as the child's error today, and that the line carries a fixed prefix so it never begins with `/`.

### [MEDIUM] D4's arms, pause seam and probe rate

**Location:** `_lyx/discussion/decision-record.md:66` (D4).
**Issue:** The Bound says `lyx batten pause` is honoured within one check, but the pause flag is in batten's own status file, a path `battenshed` is not given today, and the Constraints list the injected seams without it.
For a running child whose driver strand is dead and whose status file does not change, no arm event or state change ever fires: the wait is bounded only by the one driver-dead notice, which D4 should say.
"A status read fails (a hard error)" does not say what a failed `stat` of the child's status file does, and the child's status is replaced by rename.
"Notices are evaluated on every check" makes a driver-liveness query and an activity read every 2 s for each batten, against one per 30 s before.
**Fix:** Name the own-status seam and add it to the Constraints.
Say a failed stat reads as unchanged for that check and a decode failure is the hard error.
Say the notice probes run at most once per a fixed notice-check interval, longer than the 2 s check, and that the driver-dead notice is the only signal for a dead driver with an unchanged status file.

### [LOW] D4 and the Constraints on exempt Stuck

**Location:** `_lyx/discussion/decision-record.md:145` (Constraints), `_lyx/discussion/decision-record.md:70` (D4), `contracts/recipes/batten-recipe.yaml`.
**Issue:** The Constraints say "shedengine's bounce and exempt-Stuck semantics are unchanged", and they are: the engine is untouched.
What changes is that no row Stuck is counted any more, so the bounce budget stops bounding the row, which the Constraint does not say and D4 states only as the removal of the 12-hour window.
D4 says the `max_bounces: 1440` rationale goes but not whether the key goes; the header comment warns that a dropped key inherits the engine's budget of ten, which stays harmless only while every Stuck is exempt.
**Fix:** In the Constraints, say the engine is unchanged and the row no longer produces a counted Stuck.
In D4, say the key stays or goes, and that its removal is safe only because no Stuck is counted.

### [LOW] D2 is not needed by D1

**Location:** `_lyx/discussion/decision-record.md:49` (D2 Rationale), `internal/shuttleengine/claudeengine/usage.go:102`.
**Issue:** `CompactedSince` reads the turn end only for its transcript path and, for `ReadTurnEndAfter`, its message; D1 ignores that field and re-reads `ContextTokens` at completion.
The turn end held from the idle phase names the same transcript, so both D1 criteria (a boundary with a later turn end, a boundary with none) and the restart criterion hold without D2.
The rationale "it must name a current turn end" is not true of the mechanism.
**Fix:** Remove D2 and the "which D2 keeps current" clause in D1, or state what in D1 or an acceptance criterion needs a current `ReadingTurnEnd`.
The record keeps D2 only if it names that need.

### [LOW] D7 and one channel per event

**Location:** `_lyx/discussion/decision-record.md:124` (D7 Rationale), `_lyx/discussion/decision-record.md:79` (D5).
**Issue:** The Rationale says one channel per event, but D7 keeps the driver's `parent_notice` relay for an `awaiting` envelope, and D5 lists awaiting among the states batten notifies.
An awaiting child produces both messages, the double message D7 set out to remove.
**Fix:** Say which of the two an awaiting child sends: either batten's notice carries the parent notice and the relay is dropped, or the relay stays and the awaiting state is left out of the state-changed notice.

### [LOW] Two assumptions that are not assumptions

**Location:** `_lyx/discussion/decision-record.md:157` (A1), `_lyx/discussion/decision-record.md:164` (A8).
**Issue:** A1 is a finding from a transcript, already in D1's Rationale and the support log's Interview and Question ledger.
A8 restates D3's Rationale.
Neither is a guess the plan writer must hold open, and neither carries a requirement.
**Fix:** Remove both from `## Auto-mode assumptions`.
Nothing is lost: the content stays in D1 and D3.
