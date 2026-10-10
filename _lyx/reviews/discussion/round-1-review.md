---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md (D1 Bound; A1; R1)
    summary: D1's Bound rests on a false premise, since the Bouncer's continue Stuck after a budget escalation is BudgetExempt, so a gate-failed Burler round after a budget continue cycles with no counted Stuck on either row
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md (D4 Decision; Notes for the plan writer, D4/D5 bullet)
    summary: D4 sources the turn start from the shadow fold, which runs after pollEventsTick in each tick, so a turn start and turn end in one batch drop the task turn end for good, and the loop then branches on the shadow the package doc and D5 say it never acts on
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D4 Bound; A4; R3)
    summary: D4's Bound covers a session that never starts a turn, but not a provider that runs turns without writing a turn-start line, which silently drops every hold and its notice until the deadline; that bypass is stated only in R3
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D3 Decision; AC7, AC8)
    summary: D3 does not say the wall-clock delta must strip Go's monotonic reading, without which two time.Now readings subtract to the awake delta and no gap is ever seen, and every acceptance criterion runs on fakes that cannot catch it
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md (D3 Decision)
    summary: D3 says the api-error check and the no-live-run fallback judge now.Sub(newest) less the gap accumulation, but the api-error check judges each run's own LastActivity and the fallback's newest is a history or episode stamp, so the accumulation's input and reset key are undefined for both
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md (D5 Bound)
    summary: D5's display-only Bound holds but is asserted, not shown; the held marker is read by batten's activity reading and loom's idle activity, and the Bound does not say they treat held and absent alike
  - id: F7
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md (D3 Decision, last sentence)
    summary: The notice text keeps reporting the awake duration reads as if today's text reports awake time, while today it reports the wall delta
---

### [BLOCKING] D1's Bound rests on a false premise: not every Bouncer Stuck is counted

**Location:** `_lyx/discussion/decision-record.md`, D1 Bound, A1 and R1.

**Issue:** D1's Bound says "every Stuck the Bouncer row returns, a seed, a CONTINUE and a degraded judge exit alike, is counted on the Bouncer row".
That is false: `settleUnconverged` in `internal/shedadapters/bouncer.go` sets `ptr.BudgetExempt = cause == EscalationBudget` on the Stuck it returns for a recorded continue decision, and `episodeStuckCount` in `internal/shedengine/run.go` skips exempt entries.
The record also leaves the Burler's third Stuck, the gate-failed exit in `doneExit` (empty Path, round archived), unnamed: D1 names "a completed round and an unjudged-round hand-back", while its Bound says the Burler row's engine budget "no longer caps anything".

Traced from the code, after an operator `lyx loom circling continue` on a budget escalation at round N:

1. The Bouncer returns an exempt Stuck to the Burler.
2. Round N+1's gate fails; `doneExit` archives the round and returns Stuck with an empty Path.
3. The Bouncer resolves round N again (N+1 was archived), finds it judged, calls `settle`, reads round N's continue decision with cause budget, and returns another exempt Stuck.
4. The Burler runs round N+1 again.

Today step 2's Stuck is counted on the Burler row, so its engine budget ends the cycle.
Under D1, if the gate-failed Stuck is exempt as the Bound implies, nothing in the cycle is counted on either row, and each turn of it pays for a full review and fix round.
If the gate-failed Stuck stays counted, the Bound's "no longer caps anything" is false and the Burler budget's leftover-generation problem the Decision removes survives for gate failures.
Either way, the plan writer cannot tell which, and A1 and R1 do not mention the exempt continue path.

The unjudged-round ping-pong claim does hold: that cycle passes through `judgeCall`, whose failures go through `degrade`, which never sets `BudgetExempt`.
The record asserts this rather than showing it, which is what let the false generalization above through.

**Fix:** In D1, state which Burler Stucks are exempt, naming the gate-failed exit explicitly.
Correct the Bound: name the Bouncer's continue-after-budget Stuck as exempt, and state what bounds a gate-failed round after a budget continue (for example, the gate-failed Stuck stays counted on a bound that resets per continue decision, or the Bouncer counts a gate-failed hand-back over a decided round).
Add an acceptance criterion that pins that cycle halting.

### [BLOCKING] D4 reads the turn start from the shadow, which folds after the hold decision

**Location:** `_lyx/discussion/decision-record.md`, D4 Decision ("the session signals the loop already folds from the prompt offset") and the D4/D5 bullet of Notes for the plan writer.

**Issue:** In `Wait` (`internal/shuttleengine/wait.go`), each tick calls `pollEventsTick` first and `logSessionState` afterwards; only `logSessionState` advances the shadow cursor and folds turn starts.
`pollEventsTick` advances `run.offset` past every event it parses and returns the held turn end in the same call.
So when a turn start and a turn end arrive in one batch, the hold decision runs before the shadow has folded that turn start, D4 classifies the task's turn end as pre-prompt, logs it at Info and drops it, and the next tick never re-reads it.
The stalled agent is then neither held nor notified until the deadline, contradicting D4's own Bound ("After that start every turn end counts as today").
One batch holding both is ordinary: Wait's first tick after `Start` returns, a slow tick, and an attached Wait, whose `run.offset` may start well past `PromptOffset` while the shadow cursor starts at `PromptOffset`.

The sourcing also contradicts the package's contract: `waitshadow.go` states "the loop never branches on it, and a failure to read it changes nothing", and D5 adds a package-doc line saying the session state is a shadow the loop never acts on.
The support log's rejected alternative for fault 5 says "the loop reads the turn-start signal directly instead", which disagrees with D4's text and the Notes.
AC10 does not cover a turn start and turn end in the same batch.

**Fix:** Decide in D4 that the hold decision reads turn starts itself, from the same bytes `pollEventsTick` parses, ordered by offset within the batch, independent of the shadow, and that a shadow read failure still changes nothing.
Bring D5's turn-start read under the same rule.
Add a same-batch row and an attached-Wait row to AC10.

### [MEDIUM] D4's Bound omits a provider that writes no turn-start line

**Location:** `_lyx/discussion/decision-record.md`, D4 Bound, A4 and R3.

**Issue:** The Bound covers "a session that never starts a turn after the prompt", but the case R3 names is different: the session runs turns, the provider writes no turn-start line the parser recognises, and D4 then skips every turn end, real ones included.
Every hold and every hold notice is lost, each skipped turn end is logged only at Info, and the run waits out its deadline.
That is a guard made bypassable in the other direction, and the rubric's attack-surface rule wants what it lets through and what bounds it stated in the Decision, not only in Open risks.

**Fix:** State this bypass in D4's Bound and bound it, for example by logging the skip at Warn when no turn start has been read past `PromptOffset` within a stated window, or by holding a turn end once the run has seen a second turn end with no turn start between.

### [MEDIUM] D3 does not say the wall delta strips the monotonic reading

**Location:** `_lyx/discussion/decision-record.md`, D3 Decision; AC7 and AC8.

**Issue:** D3 compares "the wall-clock delta of `Now`" across probe checks with an awake delta read from Go's monotonic clock.
Go's `Time.Sub` uses the monotonic reading whenever both operands carry one, so with `deps.Now` resolved to `time.Now`, the "wall" delta between two checks is itself the awake delta, no gap is ever observed, and D3 is a silent no-op in production.
AC7 and AC8 drive fake clocks whose times carry no monotonic reading, so every criterion passes against that broken wiring.

**Fix:** State in D3 that the wall delta is computed from readings with the monotonic part stripped (`Round(0)` or `UnixNano`), and add a criterion that exercises the production `Now` resolution, or pins the stripping at the seam.

### [MEDIUM] D3's subtraction input does not match the api-error check or the fallback

**Location:** `_lyx/discussion/decision-record.md`, D3 Decision.

**Issue:** D3 says the quiet notice, its no-live-run fallback and the api-error check all judge `now.Sub(newest)` less the accumulation, and that the accumulation drops when "the newest activity moves".
In `judgeAgents` (`internal/battenshed/notice.go`) the api-error check judges each erroring run's own `LastActivity` against `noticeAPIErrorIdle`, not the newest activity, and the fallback's `newest` is the later of the newest history entry and the episode's since, not agent activity.
Read literally, D3 moves the api-error check onto the newest activity, which AC8 forbids, and it leaves undefined what resets the accumulation while no live run exists.

**Fix:** Name in D3 the stamp each of the three checks subtracts from, and the stamp whose movement resets the accumulation for the fallback.

### [LOW] D5's display-only Bound is asserted, not shown

**Location:** `_lyx/discussion/decision-record.md`, D5 Bound.

**Issue:** The held label reaches Go decisions outside shuttle: `agentActivityFor` in `internal/loomcli/looparm.go` and the activity reading in `internal/battencli/wire.go` both read wait markers and skip a held one.
The Bound holds, because both treat a held marker and an absent one alike (each falls through to the file-time reading), and the held wait ranks lowest, so ending it reveals no other wait.
The record says none of this, so the plan writer cannot check that the earlier clear keeps that equivalence.
The other edge the focus names, a turn start after a held turn end that was never notified, is not a defect: `notifyHeld` sends at the hold or never (interactive, quiet hold, a failed persist) and has no retry an earlier clear could cancel.

**Fix:** Add to D5's Bound that the held marker's readers treat held and absent alike, so clearing it earlier changes no activity reading.

### [NIT] "keeps reporting the awake duration"

**Location:** `_lyx/discussion/decision-record.md`, D3 Decision, last sentence.

**Issue:** Today's notice text reports `now.Sub(newest)`, the wall delta; "keeps reporting" implies it already reports awake time.

**Fix:** Write "the notice text reports the awake duration".

## Focus departures

None.
Verdicts on focus items that produced no finding:
D2's prose claims match the engine (`episodeStuckCount` ends the gate's episode at any `PR-Review` row's Done).
D3's tolerance, reset and Windows behaviour are stated in the Decision's own Bound, apart from the gaps in F4 and F5.
The relocation check found no deliberation in `decision-record.md` that should move: A5, A6 and R4 overlap the question ledger, but each carries a requirement or a bound the plan writer needs.
Every Decision has a criterion that fails on regression, apart from the gaps named in F1, F2 and F4; none is untestable as written.
