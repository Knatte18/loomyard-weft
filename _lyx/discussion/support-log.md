# Support log: loop-small-bugs

## Interview

Autonomous mode: no operator answered; every question below was settled by the Discussion producer and recorded in the question ledger.

1. Read the board entry (five faults, issues #503, #526, #485) and the entries that touch it: `webster-stability` (shuttle turn ends and gate shells, no overlap with the hold check), `reed-hygiene` (watchdog wakeups, no overlap), `shuttle-color-verified-send` (the launch line carrying the first turn; D4 keeps the prompt offset the authority so that later change needs no rework), and `lyx-halt` (records the same sleep-time idle notices as motivation; D3 fixes the notice independently of any pause verb).
2. Explored `internal/shedengine` (budget check after a Stuck, episode counting with segment enders and goto), `internal/shedadapters` (Burler hand-back Stuck, `roundBudgetExempt`, Bouncer `budgetReached` and escalation), `internal/shedengine/progress.go` (`Bounces` counts the main-line row), the loom and darn recipes and the overview sentence on `lyx loom reject`.
3. Explored `internal/battenshed/notice.go` (`judgeAgents` compares `deps.Now()` with file-time activity) and `deps.go` (the `Now` and `Sleep` seams).
4. Explored `internal/shuttleengine`: `loadSkillsThenPrompt` persists `PromptOffset` from the run's offset after the load turns and sends the prompt afterwards; `pollEventsTick` holds the last parsed event when output files are missing; the held mark clears only on a tick that parsed an event; the shadow folds session signals from `PromptOffset` and logs disagreements against a fixed mapping; `turnStartedSince` already reads turn-start signals.
5. Settled the five decisions in the record and the assumptions each rests on.

## Rejected alternatives

- Fault 1, engine-side pre-call budget check: moving the engine's budget comparison from after a Stuck to before the producer's call changes the bounce semantics of every recipe row for a fault that lives in one adapter pair.
- Fault 1, raising the Burler row's budget by one: hides the two-row offset behind a magic number and still blocks an approved round whenever an extra hand-back occurred.
- Fault 1, counting both rows in `budgetReached` and taking the maximum: keeps two enforcement points and still lets the engine block the Burler before the judge.
- Fault 1, exempting the gate-failed Burler Stuck too and counting a gate-failed hand-back over a decided round on the Bouncer: moves a gate failure's bound into the Bouncer's decision path for a case the Burler row's counted Stuck already bounds.
- Fault 2, deleting `max_bounces: 5` from the PR-Gate rows: the status projection and the reject refusal would then report the engine default, a visible change the board entry does not ask for.
- Fault 2, building a per-task rejection budget on the gate: ruled out by the board entry.
- Fault 3, measuring idle time as awake time since batten first observed the current newest activity: delays a notice for a run that was already idle when batten started, and drifts by a probe interval.
- Fault 3, detecting sleep as a wall-clock delta across one `Sleep` that exceeds the interval by a tolerance: needs no seam but misses short sleeps, depends on the check interval and cannot tell sleep from a long probe.
- Fault 3, a suspend-aware clock in every unbounded wait (shuttle, orch watcher, reed watchdog): outside the board entry, which names batten's idle notice only.
- Fault 4, only moving `PromptOffset` to the events size right before the prompt send: covers a line appended before the send but not one appended after it; kept as a companion to the turn-start rule, not as the fix.
- Fault 4, reading the turn start from the shadow's fold: the shadow folds after the hold decision in each tick, so a turn start and turn end in one batch would drop the task turn end, and the loop would act on the shadow.
- Fault 4, bounding a silent provider by holding the second turn end read with no turn start between: a stall at the first task turn end still waits out the deadline; arming the guard only on a turn start before `PromptOffset` keeps such a provider on today's behaviour.
- Fault 4, filtering turn ends by their message text matching the skill-load reply: provider-specific and fragile.
- Fault 5, clearing the held mark from the folded session state: makes the loop act on the shadow, which the package doc rules out; the loop reads the turn-start signal directly instead.
- Fault 5, stopping the run on a lasting disagreement: the board entry asks for resolution to one state, and the session state is a shadow reading.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick). Where should the review budget be enforced? Picked: the Bouncer row for judged rounds, with the Burler's judged and unjudged hand-backs exempt and its gate-failed exit still counted (D1). Rationale: the Bouncer already checks before spawning a round, and the Burler's count was the only path to a blocked state on an approved round.
- Q2 (auto-pick). Keep or delete `max_bounces: 5` on PR-Gate? Picked: keep, reword the prose (D2, A2).
- Q3 (auto-pick). Which sites carry the false claim? Picked: the loom recipe header, the darn recipe header and the overview's reject sentence; AC5 asks the Plan to grep for more.
- Q4 (auto-pick). Monotonic seam or sleep-delta heuristic for the idle clock? Picked: a monotonic awake-clock seam with a one-minute gap tolerance (D3). Rationale: exact while awake, testable through `deps`, no new wake.
- Q5 (auto-pick). Apply the awake measure to the api-error idle check too? Picked: yes, through the same helper, for one rule per package.
- Q6 (auto-pick). Mechanism of #485? Not determinable from this worktree (the `bolt-and-store` pair is gone). Picked: fix the class with the turn-start precondition and move the prompt offset to the send (D4, A5).
- Q7 (auto-pick). Which side wins a lasting `held` beside `busy`? Picked: the loop, which now leaves `held` at the next turn start, and the log line says so (D5).
- Q8 (auto-pick). Should the orch watcher's matching disagreement line change? Picked: out of scope; the board entry names shuttle's line.
- Environment: `lyx board`, the codebase and `lyx loom validate-discussion` were usable; nothing outside the two discussion files was touched.
