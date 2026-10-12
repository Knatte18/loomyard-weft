---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:118
    summary: D7 states the tackler's read-only, mutation-free bound as a fact, but nothing enforces it beyond stencil text, and the tackler's resume is not tied to the arbiter's under-lock Admit, so a wrong or stale verdict can move a run another actor already moved
    fix: "In D7, replace the sentence It runs no lyx verb that changes the run and no mutating git command with the enforced bound: (a) the no-mutation rule is stencil instruction only, as the arbiter's is, and Go checks after the tackler ends that the run's stop key and the worktree's uncommitted paths are unchanged from spawn time, reading a change as hand-back with cause tampered; (b) a tackler resume is performed through the same consume path as an arbiter resume (shedengine.Relaunch with an Admit that, under the status lock, refuses unless the stop key and pause conditions equal those the attempt recorded at spawn, and writes the applied mark in the same hold), a refusal acting as a new stop per D12; (c) a wrong resume or goto is bounded to one re-step or one move to a row at or before the current row per attempt, at most tackler_max_spawns per run, and a returning stop key goes to the arbiter row of D3. Add an acceptance criterion: a tackler resume whose stop key changed between spawn and consume performs nothing and is handled as a new stop."
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:69
    summary: D4's verify-failure goto matches a record only by row and target, weaker than batten's recordMatchesHalt, and its one-retry cap lives in Go settle records while batten's lives in its publish-retry marker, so a stale record can drive a goto and a run can be retried twice across a watched/unwatched change
    fix: "In D4, state that the Go driver accepts the verify failure record only under the same match batten's publish retry uses: a target and write time present, the record's head equal to the child's HEAD, the halting entry a Stuck of the record's row, and the write time after the previous history entry and at or before the halting one. State that the one retry per run is spent if either batten's publish-retry marker or a Go settle record shows a retry, and that the Go driver writes or honours batten's marker state so batten adopting the run later does not retry again. In D3, reword the last row's clause to any other blocked stop, so a blocked stop whose record fails the match lands on the tackler row instead of matching no row. Add acceptance criteria: a stale record from an earlier halt causes no goto; a run whose retry batten spent is not retried by an unwatched Go driver, and the reverse."
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:94
    summary: D5 gates both hand-over edges on the guide's clean-tree guard, which no stencil or code defines, and leaves a crash between releasing the driver lock and spawning the fresh driver with no stop report, park marker or notice
    fix: "In D5, define the clean-tree guard in the decision itself: the task worktree has no uncommitted or untracked path outside the run's own records directories, read through the existing dirty-paths read, and a dirty tree hands back with the paths in the stop report. State the order of the hand-over: write the hand-over settle record and the stop report with the park marker first, then release the driver lock, then spawn; the fresh driver removes the marker on start (D2). State the recovery bound for a crash or failed spawn after release: the run stays parked with a stop report, batten's InnerRun condition re-steps it when watched, and when unwatched the parent receives the D8 unwatched hand-back line. State where batten's once-per-episode-and-build cap is recorded so a batten restart does not reset it. Add an acceptance criterion: a hand-over whose spawn fails leaves a parked run with a stop report."
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:38
    summary: Under D1 a --child-driver typed against an adopted prime seed that records no child_driver is neither recorded nor refused, so driver_kind silently overrides the typed flag, breaking D1's precedence and the loud-failure intent of PATTERN-driver-choice-single-site
    fix: "In D1, decide the adopted-seed case: refuseAdoptedSeed refuses a typed --child-driver against a found seed that records no child_driver, naming the way forward (delete the seed to re-seed, or drop the flag to take driver_kind), with its refusal-spec row. Add the case to the Seeding acceptance criterion and to the refusal list in Constraints."
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:171
    summary: D13 treats an empty-reason paused stop as a probable shed bug, but shed persists paused with no reason by design after a goto, a bare pause request and a cancellation, so D12's moved-run path writes a friction note at every outside goto
    fix: "Rewrite D13: the empty-reason paused stop is the designed outcome of shedengine's goto, a bare pause_requested pause and a context cancellation, none of which records a reason. Decide that shed records a reason at each of those persists (goto to the target, pause requested, cancelled) so every paused stop carries one, and that the Go driver writes the friction note only for a paused stop whose reason is still empty after that change. Update the matching acceptance criterion and the Scope line to say the same."
  - id: F6
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:71
    summary: D3 re-steps a transient error twice more before the tackler, while the LLM driver sends a transient that reaches it, already re-stepped once by the loop, straight to the arbiter; the Notes say the table mirrors the LLM driver and no decision names this departure
    fix: "In D3, either change the transient error row to Tackler (D7), budget permitting, matching the LLM driver's no-further-re-step rule, or keep the re-steps and add one sentence beside the interrupted-under-handback note stating that a transient reaching the Go driver has already been re-stepped once by the loop and the Go driver re-steps it up to the repair cap before the tackler, unlike the LLM driver. Change the Notes sentence the rule table mirrors to name the departures."
  - id: F7
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:85
    summary: D4 and D8 key behaviour on the batten-watched marker naming a live pid, a pid-only check with no start time, so a reused pid reads as watched and a verify halt is handed back with no retry and, under the watched rule, no parent message
    fix: "In D4, state the bound of the watched read: a watched reading whose pid is not a batten process (or whose recorded start time differs) is unwatched, or, if the existing pid-only read is kept, state the consequence (a stale marker parks the halt with a stop report and no notice) and add that the Go driver sends the unwatched hand-back line whenever it hands back a verify halt as watched, so a false watched reading still reaches the parent."
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:79
    summary: Decision prose carries deliberation already recorded in support-log.md, namely D3's justification of the interrupted-under-handback narrowing and D6's comparison with arbiter-when's proposed trigger, whose requirements the D3 table rows already state
    fix: "Delete the sentence at decision-record.md:79 (the D3 table rows for interrupted already carry the rule, and the support log's rejected alternative carries the reason) and delete the sentence at decision-record.md:109 (D3's last row already routes a blocked stop with no target to the tackler). If the arbiter-when comparison is wanted, add it to the support log's Interview entry for the board-fit lens."
  - id: F9
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:151
    summary: D10 names only the refusal branch's text in resume's Long help as changing, but the Long help opens by saying resume never spawns a driver, which D10 makes false
    fix: "In D10, change That refusal branch's text in resume's Long help to resume's Long help (its opening statement that resume never spawns a driver, and the lyx loom start refusal branch)."
---

### [BLOCKING] D7's read-only tackler is an instruction, not an enforced bound

**Location:** `_lyx/discussion/decision-record.md:118`, with D7's verdict handling at lines 121 and 124.

**Issue:** D7 says the tackler "runs no `lyx` verb that changes the run and no mutating git command".
The arbiter's equivalent rule is stencil text alone (`contracts/stencils/loom/loom-template-arbiter.md`, "What you never do"), and the tackler is a cheap model spawned as an interactive strand in the task worktree with the same tools.
What does bound the arbiter's `resume` is `consumeArbiterAttempt` (`internal/loomcli/arbiterconsume.go`): it relaunches through `shedengine.Relaunch` with an Admit that refuses under the status lock unless the stop key and pause conditions still equal the attempt's, and it writes the `applied` mark in the same hold.
D7 copies the `applied` mark but not that Admit, and D12's status re-read happens outside the status lock, so a tackler `resume` (or its `goto`) can land on a run batten, the orch or the tackler itself moved after the re-read.
The rubric's attack-surface rule requires the Decision to state what a wrong verdict can make Go do and what bounds it.
D7 leaves that unstated: a `goto` is limited only to "at or before the current row", up to `tackler_max_spawns` times per run.

### [MEDIUM] D4's goto trusts a weaker record match and splits the one-retry count

**Location:** `_lyx/discussion/decision-record.md:69`, D4 at lines 82–89.

**Issue:** Batten's publish retry accepts a record only through `recordMatchesHalt` (`internal/battenshed/publishretry.go`): head equal to the child's HEAD, halting entry a Stuck of the record's row, and a write time bounded by the previous and halting history entries.
A record from an earlier halt can survive, since `clearFailure` runs only on a later pass.
D4 and the D3 row ask only for a record naming the halting row and a target, so a stale record can drive the Go driver's one rule-driven goto.
The retry is counted twice over, from Go settle records (D4) and from batten's `-publish-retry` marker.
When a batten that spent the retry dies and the run is restarted unwatched, or when batten adopts a run whose retry Go already spent, the run is retried a second time.
A blocked stop whose record fails the match is covered by neither the record row nor the last row's "no failure record target", so it matches no row.

### [MEDIUM] D5 rests on an undefined guard and leaves a crash window in the hand-over

**Location:** `_lyx/discussion/decision-record.md:94`, and line 97.

**Issue:** "The guide's clean-tree guard" appears in `shed-template-driver.md` and `shed-template-driver-guide.md` only as a name; neither defines it, and no Go code implements a guard by that name.
Both D5 edges, and the acceptance criterion "a dirty tree blocks it", depend on it.
The hand-over releases the driver lock, then spawns.
That order is forced, since a held lock makes the fresh `drive` refuse (D2).
But a hand-over is not a hand-back, so D9 writes no stop report or park marker.
A crash or a failed spawn in that window leaves a halted, unparked run.
An unwatched run then has no actor and no parent message.
Batten's once-per-episode-and-build cap has no stated record, so whether it survives a batten restart is open.

### [MEDIUM] A typed --child-driver is silently dropped on an adopted seed that records none

**Location:** `_lyx/discussion/decision-record.md:38`.

**Issue:** `armSeed` never rewrites a found seed, and D1 makes `refuseAdoptedSeed` pass a seed that records no `child_driver`.
A typed `--child-driver go` against such a seed is therefore neither recorded nor refused, and Seed-Child resolves `driver_kind` (default `llm`).
That contradicts D1's own precedence ("an operator-typed `--child-driver` ... then the resolved `driver_kind`").
The carve-out in `PATTERN-driver-choice-single-site` exists to prefer a loud command-line failure to a silently discarded typed flag.
Today `battenChildDriver` writes `llm` when the flag is empty, so the no-record seed is new with this task.

### [MEDIUM] D13's premise: empty-reason paused is by design

**Location:** `_lyx/discussion/decision-record.md:171`, with the acceptance line 240.

**Issue:** `shedengine` persists `StatePaused` with no reason in `Goto` (`internal/shedengine/goto.go`, the `cur.State = StatePaused` update), on a bare `pause_requested` or a cancelled context (`internal/shedengine/run.go`, step 3), and on a producer error that coincides with a cancellation.
Only a fired `pause_before`/`pause_after` condition carries a reason.
The board's "look like a shed bug" is a hypothesis the code does not support.
Combined with D12, every goto another actor performs while a Go driver waits returns as an empty-reason `paused` stop.
D3 hands that stop back and D13 files a friction note for it, so the reflection receives a note for every routine orch goto.

### [MEDIUM] D3's transient row departs from the LLM driver without saying so

**Location:** `_lyx/discussion/decision-record.md:71`, and the Notes line 245.

**Issue:** `shed-template-driver.md` says Go already re-steps a transient once inside the loop, so a transient reaching the driver "has failed twice at the same producer: take it to `## Arbiter`".
D3 instead re-steps it up to the repair cap (two more), then sends it to the tackler.
D3's own binary-eligible definition, "a transient that failed twice", assumes the LLM driver's reading.
D3 names its `interrupted`/`handback` narrowing and D6 names its `blocked` routing, but nothing names this departure, and the Notes claim the table mirrors the LLM driver.

### [LOW] The watched read is pid-only

**Location:** `_lyx/discussion/decision-record.md:85`.

**Issue:** `liveBattenPID` (`internal/loomcli/driverprompt.go`) reads a pid and checks only `proc.IsAlive`.
After a batten dies and its pid is reused, the marker reads as watched.
Under D4 the Go driver then leaves the halt to a batten that does not exist, and under D8's watched rule a verify hand-back sends no parent message.
The LLM driver shares the read, but D4 adds a new behaviour gated on it, and the focus asks for its bound under a stale marker.

### [LOW] Deliberation in decision prose

**Location:** `_lyx/discussion/decision-record.md:79` and `:109`.

**Issue:** Line 79 justifies a narrowing that D3's table rows already encode, and the support log's rejected alternative "Sending `interrupted` under `handback` to the tackler" carries the same reason.
Line 109 restates D3's last row and adds a comparison with another board entry's proposal.
Neither carries a requirement absent from Decisions, so the completeness-before-leanness test passes and both sentences are relocatable.
D15's departure sentence stays: it tells the plan writer that `orch-split` routing does not bind go-driven runs.

### [NIT] Resume's Long help changes more than D10 says

**Location:** `_lyx/discussion/decision-record.md:151`.

**Issue:** `internal/loomcli/resume.go`'s Long help begins "resume wakes a halted run's live, parked driver ... It never spawns a driver".
D10's spawn branch makes that sentence false, but D10 names only the refusal branch's text as changing.

## Focus verdicts not carried by a finding

On the D10 directive: the LLM default holds at both seeding sites D1 names, and the resume spawn is bounded by its no-strand, no-run-lock, no-driver-lock condition, the unchanged pause and awaiting refusals, and `drive`'s own refusal over a held driver lock, so two racing resumes still yield one driver.
The one breach found there is F4.
On the D3-consistency directive: apart from F2's unmatched `blocked` case and F6, row order and first-match hold.
The binary-eligible row precedes the repair-cap and tackler rows by design.
A stop key returning after a tackler `resume` reaches the arbiter row only once the transient and `reinvoke` rows no longer match, which holds past the repair cap.
The three arbiter spawn cases agree between D3, D6 and D7.
On the D2/D11/D12 directive: the pause hold and the tackler deadline agree, since the deadline excludes paused intervals.
The liveness reads agree across start, resume, batten adoption and the driver-dead notice, since the driver lock is an flock and is released on a crash.
The left-behind state found is F3's hand-over window.
