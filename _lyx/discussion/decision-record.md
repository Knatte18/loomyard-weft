# Decision record: loop-small-bugs

## Goal

Fix five small faults in the loop machinery, each in the module that owns it:
the review bounce budget blocking an approved round (#503),
the PR-Gate budget comment stating behaviour the engine does not have (#526),
batten's idle notice counting machine sleep as idle time,
shuttle's hold notice firing on the skill-load turn end (#485),
and shuttle's wait loop reading `held` beside a session state of `busy` for a whole turn.

## Scope

In:

1. `internal/shedadapters`: the Burler round producer's hand-backs for judgment stop spending the segment's bounce budget, so the Bouncer's own pre-round check is the only place the budget on judged rounds is enforced, and the Burler row's budget caps gate-failed rounds alone.
2. `contracts/recipes/loom-recipe.yaml`, `contracts/recipes/darn-recipe.yaml` and `docs/overview.md`: the PR-Gate budget prose states the engine's real episode rule.
3. `internal/battenshed`: the quiet and api-error idle clocks measure awake time, dropping the time the machine spent suspended.
4. `internal/shuttleengine`: a turn end that precedes the start prompt's own turn start is never a held task turn end, and the prompt offset is taken right before the prompt is sent.
5. `internal/shuttleengine`: the held wait ends at the turn start that follows the held turn end, and the disagreement log line says the loop's classification stands.

Out:

- A per-task PR-Gate rejection budget (the board entry rules it out).
- Suspend awareness in waits other than batten's idle notices: shuttle's turn and shell timeouts, the orch watcher's probes and the reed watchdog keep wall time.
- A hub-wide pause or any sleep-safe stop (board entry `lyx-halt`).
- Changing how review rounds are counted or displayed in `lyx loom status`.
- The orch watcher's own `session state disagrees with the idle probe` line.

## Decisions

### D1. The review segment's budget is the Bouncer row's alone

Decision: the Burler round producer marks the two Stucks that hand a round to the Bouncer for judgment, a completed round and an unjudged-round hand-back, as `BudgetExempt`.
The third Burler Stuck, the gate-failed exit that archives the round and hands back no review, stays counted on the Burler row.
The `roundBudgetExempt` special case, which exempted only the round after a budget continue decision, goes with its decision-file read.
The Bouncer's `budgetReached` check at a CONTINUE verdict, which runs before any new round is spawned, becomes the single enforcement point of the budget on judged rounds.
The Burler row's budget caps gate-failed rounds alone: its episode ends at the segment's Done or a goto into it, as `episodeStuckCount` already applies, so it counts the gate-failed rounds of one generation.
The doc comments on `BurlerProducer`, `NewBouncer`, the Bouncer's settled-generation clear and the `bouncer_clear_test.go` comment that describe the two-row offset, a Burler episode that never resets and the "accepted limitation" of a later generation are rewritten to the new rule.

Rationale: a Burler round only ever ends with a hand-back, so counting that hand-back against a budget means a round that was approved by its reviewer and passed its verify gate can block before the Bouncer judges it, which is what #503 hit.
The Bouncer already counts judged rounds through `Routing.Bounces` and escalates with cause budget before spawning the next round, and `lyx loom circling accept` and `continue` are built on that Awaiting.
Dropping the Burler's count removes the only path that reaches a blocked state on an approved round.

Bound: the exemption lets a judged or unjudged round reach the Bouncer without spending the Burler row's budget, and nothing else.
Every Bouncer Stuck is counted on the Bouncer row except the one the Bouncer returns for a recorded continue decision whose cause is budget, which `settleUnconverged` marks exempt; the seed, the re-bounce, a CONTINUE with no decision, a circling-cause continue and every `degrade` exit are counted.
So a judge that never recovers or a round that never converges halts within `review_max_bounces` counted Bouncer Stucks, and a budget continue buys one judged round, after which the next CONTINUE reaches `budgetReached` and escalates again.
The unjudged-round ping-pong is bounded the same way: the Burler hands back only when the highest complete round has no verdict, a round with no verdict has no decision file, so the Bouncer's next call judges it through `judgeCall`, whose failures exit through `degrade` and are counted.
A gate-failed round after a budget continue cycles back to that round's decided, exempt Bouncer Stuck, so its bound is the Burler row's counted gate-failed Stuck, which blocks the run after the row's budget of gate-failed rounds in the generation.

### D2. The PR-Gate budget prose states the episode rule

Decision: the comment block in the loom recipe that says the gate's count never resets and the budget is five rejection rounds per task is replaced by the rule the engine applies: a Done by any row of segment `PR-Review` ends the gate's episode, so PR-Rework's Done resets the count after every rework and `max_bounces` bounds consecutive gate Stucks with no rework Done between them, which the recipe never produces.
The darn recipe's matching comment is reworded the same way, since a Darn Done ends the episode too and a Darn Stuck halts the run, so its "consecutive rejections the writer failed to answer" never accumulate either.
The overview's sentence on `lyx loom reject` refusing after five rejection rounds is corrected to say the refusal exists and what makes it unreachable under the recipe's episode rule.
`max_bounces: 5` stays on both PR-Gate rows: it is the value the status projection and the reject refusal report, and removing it changes nothing the task asks for.

Rationale: an agent designed against the five-round bound the comment promised (#526).
The task is to make the prose true, not to build the per-task budget, which the board entry puts out of scope.

### D3. Batten's idle clocks measure awake time

Decision: `battenshed` gains one injectable clock seam beside `Now`, an awake clock that reads a duration from a clock that stops while the machine sleeps, resolved to Go's monotonic reading when nil.
Across consecutive probe checks the wait compares the wall-clock delta of `Now` with the awake delta; the excess, where it exceeds a one-minute tolerance, is a sleep gap.
The wall delta is taken from readings with Go's monotonic part stripped, as `UnixNano` values or `Round(0)` times, because `Time.Sub` over two `time.Now` readings returns the monotonic difference, which is the awake delta itself and would never show a gap.
`childWait` accumulates the gaps observed since the activity stamp `judgeAgents` returns last changed: the newest agent activity while live runs exist, and in the no-live-run fallback the later of the child's newest history entry and the state episode's since.
A check whose stamp differs from the previous check's drops the accumulation and does not add its own gap, since that gap may lie before the new activity, so the accumulation only ever holds gaps from intervals that begin after the stamp.
A check that cannot read the activity keeps the accumulation and adds its gap.
Each check subtracts that one accumulation, through one helper, from its own wall delta: the quiet notice from `now.Sub(newest)`, the fallback from `now.Sub` of its stamp, and the api-error check from `now.Sub` of each erroring run's own `LastActivity`.
The accumulation holds only gaps after the newest activity, which no run's `LastActivity` follows, so it never exceeds the suspended time behind any of the three deltas.
The notice text reports the awake duration.

Rationale: `LastActivity` is a file time, so the only clock that can tell sleep from idleness is the process's own monotonic reading, which on Linux and macOS stops during suspend.
Subtracting observed gaps keeps today's exact behaviour while awake, including an idle run found idle at batten start, and only removes suspended time.

Bound: on a platform whose monotonic clock counts suspend, no gap is ever observed and the notice behaves as today; the seam lets a platform reader replace it later without a design change.
The one-minute tolerance means a sleep shorter than a minute still counts as idle, and a clock adjustment under a minute is never mistaken for sleep.
The gap is only ever subtracted, so the fix can delay a notice, never raise one earlier.

### D4. A turn end before the start prompt's turn start is never held

Decision: the wait loop treats a turn end as a task turn end only when a turn-start signal lies between `PromptOffset` and that turn end in the events file; a turn end with none before it is logged at Warn with its offset and message, neither holds the run nor notifies the parent, and the loop keeps polling.
The hold decision reads turn starts itself, through the engine's `SessionSignalParser` capability, from the bytes the tick parses for events and from `PromptOffset` up to where the tick's read begins, ordered by offset, so a turn start and a turn end read in one batch are told apart by position, and a Wait attached past `PromptOffset` still sees the turn start that precedes its first read.
Once a turn start is read the guard is lifted for the rest of the Wait.
It never reads the shadow's fold: the shadow stays display only, and a shadow read failure changes nothing.
The guard applies only when the events file also holds a turn-start signal before `PromptOffset`, the load turns' own, which proves the provider writes turn starts for typed turns.
An engine without the capability, a record whose `PromptOffset` is zero and a file with no turn start before `PromptOffset` keep today's behaviour.
`PromptOffset` is also taken as the events-file size immediately before the prompt is sent, after the load turns, so anything the provider appended after the last load turn end and before the prompt lies before it.

Rationale: #485 shows a hold notice whose quoted message was the skill-load reply, so a turn end that belonged to a pre-prompt turn reached the hold path.
Whatever appended it, no turn end can belong to the task before the task's own prompt started a turn, and that start is a signal the engine already reports and the verified send already consults.

Bound: the guard skips only turn ends that precede the first post-prompt turn start.
After that start every turn end counts as today, so a stalled agent is still held and notified.
A session that never starts a turn after the prompt is ended by the run's own deadline and liveness check, exactly as a session that never ends one is today.
A provider that writes no turn-start lines at all never arms the guard, since its file holds none before `PromptOffset`.
What the guard can still let through is a provider that writes a turn start for the load turns and none for the prompt turn: every turn end of the run is then skipped, each with a Warn line, and the run waits out its deadline with no hold notice.

### D5. The held wait ends at the next turn start, and the disagreement line says the loop stands

Decision: the held wait ends at the first turn-start signal read past the held turn end's offset, not only at the next turn end or ask, so a parent's answer that starts the agent working returns the loop to `running` while the session state reads `busy`.
The loop reads that turn start itself, from the bytes the tick parses, under D4's rule and never from the shadow's fold.
The `shuttle: session state disagrees` Warn line gains a fixed field saying the loop's classification stands, and the package doc says the session state is a shadow reading the loop never acts on.
No stop or other action follows a disagreement; a pair outside the fixed mapping stays a log line.

Rationale: in `cli-surface` the loop read `held` for the whole turn that followed the parent's answer, because the events parser yields no event on a turn start, so only the next turn end cleared the mark; that is the lasting `held` beside `busy` the board entry asks about.
Ending the hold at the turn start follows what the package doc already claims and removes the only lasting pair; the field makes the log answer "which side wins" without a reader opening the doc.

Bound: the held wait is display and classification only.
Ending it earlier changes no verdict, notice or return: the hold notice went at the turn end, and the run's deadline and liveness check are untouched.
It changes no activity reading either: the held marker's readers outside shuttle, loom's idle activity and batten's wait reading, treat a held marker and an absent one alike and fall through to the file-time reading, and the held wait ranks below every other wait, so clearing it reveals none.

## Constraints

- `BudgetExempt` stays a field of `shedengine.OutputPointer`; D1 sets it only on the Burler adapter's two hand-back returns, beside the Bouncer's existing budget-continue exemption in `settleUnconverged`.
- `PATTERN-shuttle-provider-seam`: the turn-start reading in D4 and D5 goes through the provider-neutral `SessionSignal` kinds; no provider specifics enter `shuttleengine`.
- `PATTERN-completion-signal`: D4 changes no negative finish answer; `allOutputFilesExist` keeps its place.
- `PATTERN-daemon-wakeups`: D3 adds no wake; the awake clock is read on the probe checks the wait already makes.
- `PATTERN-test-speed`: the awake clock is a `deps` field a test fakes, like `Now` and `Sleep`; no test waits out a duration.
- `PATTERN-test-economy`: D1's test is an extension of the existing Burler and Bouncer tests where one covers the hand-back; D4 and D5 extend the wait-loop tests with signal fixtures.
- `PATTERN-comment-line-breaks` and the recipe comment style: the rewritten recipe comments keep their block's existing wrapping style, and Go comments use semantic line breaks.
- `PATTERN-markdown-link-integrity`: the overview edit keeps every link resolving.
- `PATTERN-refusal-way-forward`: no refusal is added; the reject verb's budget refusal and its message stay as they are.
- Docs land in the same commit: `internal/shedadapters` doc comments for D1, `internal/shuttleengine/doc.go` for D4 and D5, `internal/battenshed/doc.go` for D3, `docs/overview.md` for D2.
- Test budget: a package whose new top-level tests pass its row in `cmd/lyx/testdata/test-budget.yaml` lists that file under `Edit:`; `internal/battenshed`, `internal/shedadapters` and `internal/shuttleengine` are the rows at risk.

## Auto-mode assumptions

- A1. The Bouncer's `budgetReached` is reached on every CONTINUE verdict with no decision file before a new round is spawned, so dropping the Burler's count on judged and unjudged hand-backs leaves one enforcement point on judged rounds that runs before a round starts; a recorded continue decision bypasses it for that one round by design.
- A2. Keeping `max_bounces: 5` on the PR-Gate rows is preferred over deleting a never-tripping value, because the status projection and the reject refusal read it.
- A3. The operator's machines run Linux, where Go's monotonic clock stops during suspend; Windows keeps today's behaviour under D3's bound.
- A4. A provider that reports a turn-start signal for the load turns, which go through the same verified send, reports one for the start prompt's turn too.
- A5. The exact line that produced #485's stray turn end is not on disk any more; D4 fixes the class rather than the instance, and the Plan adds a fixture with a turn end between the load turn end and the prompt's turn start.
- A6. The `held` beside `busy` pair in `cli-surface` was the turn after a parent answer; D5 fixes that cause and leaves other transient pairs as log lines.

## Open risks

- R1. If some Bouncer Stuck path other than the budget-continue one is exempt, or some Burler Stuck other than the gate-failed exit hands back a round that does not reach the Bouncer's judge or `budgetReached`, a cycle through it has no counted stop; the Plan confirms each Stuck path of `Call`, `settle` and `degrade` against D1's Bound.
- R2. A one-minute sleep-gap tolerance is a guess; a machine that stalls for over a minute without sleeping (a frozen VM) would also subtract the stall, delaying a notice by that stall.
- R3. If a provider writes turn-start lines for the load turns and stops writing one for the prompt turn, D4 turns every hold into a wait to the deadline, visible only as D4's Warn lines; the engine's signal parser is the place to notice that.
- R4. If `cli-surface`'s `held` beside `busy` had a different cause, D5 still removes the parent-answer case and the new log field makes the next instance self-explaining.
- R5. The reject verb's budget refusal becomes documented as unreachable under the recipe; a future per-task budget reopens it.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: converged; each finding below was addressed by the round's fixer, and no fresh reviewer has seen the fix.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - d3-stamp-change-drop-versus-add-order-open (class decision, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- AC1. A Burler round whose reviewer approved and whose fixer passed its gate, handed back as the Nth round at budget N, reaches the Bouncer's judge and, on CONVERGED, the segment's Done; no judged or unjudged hand-back makes the run persist `bounce budget exhausted for <segment>-Burler`.
- AC2. An unjudged-round hand-back from the Burler spends no budget, and a Bouncer that degrades on every judge call still halts the run within `review_max_bounces` Bouncer Stucks.
- AC2a. After a budget continue on round N, a round N+1 whose gate fails on every attempt is counted on the Burler row each time, and the run blocks with `bounce budget exhausted for <segment>-Burler` once the row's budget of gate-failed rounds is spent.
- AC3. A segment re-entered after a settled generation starts its Burler rounds with no leftover count binding before the Bouncer's own budget.
- AC4. `lyx loom status` reports the same count and budget for a review segment as before the change.
- AC5. No comment in `contracts/recipes/`, no prose under `docs/` and no Go comment claims a per-task rejection budget or that the PR-Gate count never resets; the loom and darn recipe comments state the episode rule.
- AC6. The reject verb's budget refusal and its message are unchanged.
- AC7. With a fake wall clock advanced by hours across one probe check and a fake awake clock advanced by seconds, a running child whose agents' newest activity predates the gap raises no quiet notice until the awake time since that activity reaches `NoticeQuiet`.
- AC8. With wall and awake clocks advancing together, the quiet notice, its no-live-run fallback and the api-error notice fire at the same checks as today.
- AC8a. The gap helper takes its wall readings with the monotonic part stripped, so the production `Now` resolution yields the wall delta; a test pins the stripping at the helper's inputs, since no test can make a real reading's two clocks diverge.
- AC8b. An erroring run whose own `LastActivity` predates a sleep gap raises the api-error notice only once its awake idle time reaches `noticeAPIErrorIdle`, and with no live run the fallback's stamp moving drops the accumulated gap.
- AC9. Newest agent activity that moves after a sleep gap drops the accumulated gap, so a later quiet period is measured from the new activity alone.
  Row: a check that observes both a sleep gap and a changed stamp subtracts that gap from no later check.
- AC10. A turn end read between `PromptOffset` and the first post-prompt turn start holds nothing, sends no notice, is logged at Warn, and the loop keeps polling; the next turn end after a turn start is held and notified as today.
  Rows: a turn start and a turn end read in one tick's batch hold the turn end; a Wait attached with its read offset past the prompt's turn start holds its first turn end; a shadow read failure changes no hold.
- AC11. An engine without the session-signal capability, a record with a zero `PromptOffset` and an events file with no turn start before `PromptOffset` hold the first turn end as today.
- AC12. `PromptOffset` equals the events-file size at the moment before the prompt is sent, so a line appended after the last load turn end and before the prompt lies below it.
- AC13. A held wait ends at a turn-start signal past the held turn end's offset: the wait mark clears, the loop classification reads `running`, and the shadow log reports no disagreement beside `busy`.
- AC14. The disagreement Warn line carries the field saying the loop's classification stands, and the package doc states it.
- AC15. `lyx gate test` over the touched packages and `lyx gate test --tags integration` over them are green.

## Notes for the plan writer

- D1's code sits in `internal/shedadapters/burler.go` (the hand-back Stuck returns and `roundBudgetExempt`) and `bouncer.go` (`budgetReached`, `settleUndecided`); `internal/shedengine/run.go`'s exempt skip and `progress.go`'s `Bounces` need no change.
- D2's prose lives in the loom recipe's header comment block, the darn recipe's header, and the `reject` sentence of the overview's loom section; grep for "never resets" and "rejection rounds" to find every site.
- D3's seam goes beside `Now` in `internal/battenshed/deps.go`, resolved in `NewInnerRun`; the comparison belongs in `judgeAgents` and the no-live-run branch of `notice.go`, with the accumulation on `childWait`.
- D4 and D5 both read `SessionSignalTurnStart` in the loop's own read, not from the shadow in `waitshadow.go`, which folds after `pollEventsTick` in each tick; the hold decision is in `pollEventsTick` and the wait loop in `wait.go`, the mark in `waitmark.go`, and the prompt-offset write in `loadSkillsThenPrompt`.
- `internal/shuttleengine/doc.go` already claims the held wait ends at the following turn start; D5 makes that true.
