## Goal

The dobby resumes a run handed back for a pause once the pause lifts, and never a run the orch holds

Takes: D4's pause-lift and hold steps: `liftPending` and `spawn` in `internal/dobbyengine/passes.go`, `syncBoltHold` in `internal/dobbyengine/catchupwait.go`, `Pause` in `internal/tenderengine/pause.go` and `pauserecord.Entry`; the `--no-wait` flag from loom-verbs-await-park, which batten passes on the lift pass's child resume.
Delivers: The lift resumes a run whose last stop is the hand-back for the lifted pause, matched by scope, with a dobby pass on a run that is not drivable, keeping the lift in the dobby state record until the resume succeeds or the run moves on; a hold reason on a pause entry, set on a live entry of the scope when one exists, that stands through a `lym tender resume` with no slug and is lifted only by one naming the slug; a bolt-hold entry carrying a hold reason replaced by the dobby's own hold entry when it lifts the bolt-hold. The driver rows of D4 are built by driver-settles-stops.
Exports: Fixed for consumers: the hold reason field on `pauserecord.Entry` and the tender pause option that sets it, which the orch's hold of orch-unattended-rules uses.

## Scope

The part changes these packages:

- internal/dobbyengine (about 480 changed lines)
- internal/tenderengine (about 220 changed lines)
- internal/pauserecord (about 50 changed lines)

## Decisions

### D4 — lym settles the mechanical classes within the table's bounds

The driver's rule table, `internal/sheddriver/driverules.go`, gains rows for the classes D2 gives lym, each bounded by the table.
Each class but the bounce budget's is raised as a named kind or field where the stop starts, so no other rule matches on message text:

- A board claim push refused at `Worktree-Create`, for a held board lock or a moved ref, gets a board sync and a re-step.
  The refusal is classified as its own class where `internal/boardengine` raises it.
- An agent launch that never became ready, a session that timed out, a failed fixer fork and a strand lost from reed get a fresh start of the row's producer.
  The runner that starts the agent raises each as its own kind where it sees the failure.
- A bounce budget exhausted at a row gets a resume, which grants one more budget as the parent's resume does today.
  The row matches the `shedengine.ReasonBounceBudgetExhausted` reason prefix, as `budgetExhausted` in `internal/sheddriver/driverules.go` and `arbiterstop.Check` do today:
  it is the one row that matches a prefix, a constant the budget's blocked arm writes.
- A Stuck after the row's gate attempts ran out gets a re-step with a fresh producer session.
  The producer gate marks the Stuck as gate attempts exhausted where it writes it.

Today a halted or paused run gets no pass until the orch or the operator resumes it with a hand pass (`internal/dobbyengine/doc.go`),
and the dobby already records each run whose covering pause lifted in `liftPending` (`internal/dobbyengine/passes.go`), which gives a pass only to a drivable run.
D4 extends that lift: once a pause lifts, the dobby resumes a run whose last stop is the hand-back for that pause, a batten step refused for a pause among them,
matched to the pause entry by its scope, the hub or the run's slug.
The lift gives such a run a dobby pass (`spawn` in `internal/dobbyengine/passes.go`) though the run is not drivable,
and the pass resumes it with D5's no-wait option on its child resume, so the lift never waits on the park.
The dobby keeps the lift in its state record until that resume succeeds or the run moves on,
so a resume refused because the driver has not parked is retried on a later tick, by a restarted dobby too.
A run that has moved on since, or that another pause still covers, is left alone.
A run whose pause the orch set as a hold (D6, D9) is never resumed by the lift.
The hold's pause entry carries a hold reason and, as a bolt-hold entry does today, stands through a `lym tender resume` with no slug;
only a `lym tender resume` naming the run's slug lifts it, and the run then waits for the orch or the operator to resume it.
A pause of a scope that already has a live entry adds none (`internal/tenderengine/pause.go`),
so a hold over a slug that already holds a live entry, the operator's slug pause or an adopt's bolt-hold entry, sets its hold reason on that entry.
When the dobby lifts a bolt-hold entry that carries a hold reason, through `syncBoltHold` in `internal/dobbyengine/catchupwait.go`, it replaces it with a hold entry of its own and never resumes the run.

The tackler keeps every stop no row names.

Bound: each row settles at most its table bound per class at the row (D3); a stop past the bound is handed back with its verdict (D7).
The dobby resumes a paused run only once its pause lifted, only from that pause's hand-back, and never a run the orch holds.

Rationale: the study shows the tackler resumed 57 of its 65 sessions; rules settle those classes without an agent and with a bound.

### Added after Discussion (parent, 2026-11-27): A stop that leaves no history entry counts once and is recorded (F37)

Decision: Adopt F37's fix: in D3, a stop that leaves no history entry (a producer error such as launch never ready or agent died) counts once too. The driver gives each stepped stop an identity that stays the same across a driver restart; every settle record of it, hand-back and arbiter restep records included, carries that identity; the count at a row is the distinct stops over the halt entries, the handed-on Awaiting entries and the identities of entry-less stops D3 does not exclude. A hand-back or arbiter restep record excludes only a stop already counted or excluded. In D7, the four facts of such a halt live in a stop record the driver writes through internal/state and dobby's child read reads, never in an earlier row's newest entry, and the stop identity is part of the halt key, so two successive failed halts at one row get different keys for D8's dedup, D7's post and D10's chase. Tier0 criteria: an agent launch that fails three times at one row with no history entry (one fresh start, a hand-back, a resume, a third failure) reaches settle: operator; a failed-state hand-back records the four facts, and a second one gets a new key and a second post.

Rationale: Without it the third halt at a row never turns severe for the commonest entry-less stops, which defeats the rule table's bound; the fix stays within D3 and D7.

### Added after Discussion (arbiter, 2026-11-27): A stop with no history entry is counted and recorded through its stop identity

Decision: The Discussion review's round-3 finding F37, that a producer error leaving the run failed appends no history entry and so is neither counted toward the third halt at a row nor recorded, is answered by D3 and D7 as they stand: D3's stop identity in the driver's stop record counts such a stop once, across a driver restart, and every settle record carries it; D7 records the four facts on that stop record and keys the halt by its row and stop identity, which D8's dedup and D10's chase read. The design needs no further change for it; the plan writer builds D3, D7, D8 and D10 and their criteria as written.

Rationale: appendHistory in internal/shedengine/run.go skips an empty outcome and StopKey in internal/shedrun/settle.go is the producer, history length and state, so the premise holds; the round-3 fix adds each part the finding's fix asked for: the identity, the narrowed nothing-count bullet, the stop record as the home of the four facts, the identity in the halt key, and the three-failure launch, second failed halt and restart criteria.

## Constraints

- PATTERN-config-layers: `beam.autonomy` stays read only through `dobbyengine`'s one lookup; its booleans keep their meaning, so no reinterpretation ships (D1).
  `beam.halt_chase_min` is a new key and needs no move table.
- PATTERN-config-key-names: every message and comment names `beam.autonomy`, `beam.halt_chase_min` and `slack.remind_after_min` in full through `configengine`'s key-name helper.
- PATTERN-shed-driver: the new driver rows sit in `internal/sheddriver`'s one rule table and stop chain; the park wait is a wait in the verb, not a new stop chain.
- PATTERN-run-fsm: the driver's and the dobby's settles act through `internal/runfsm`'s act rule; a resume past a bounce budget is the same transition the parent's resume makes.
- PATTERN-daemon-wakeups: the park wait and the chase are waits that wake at most once a second and back off while idle;
  the park wait is one an agent blocks on, so its cap comes from the keep-warm resolver; a test injects their clocks.
- PATTERN-agent-addressing: lym reaches Slack only by creel mail to `<shortname>:SLACK`, and only `internal/slackengine` calls Slack (D7, D8).
- PATTERN-refusal-way-forward: the refusal of an unknown level, the park wait's refusal past its cap and D12's warning each name a way forward a test reaches, and none names a step agents' settings deny.
- PATTERN-stencil-ownership and PATTERN-heddle-instructions: the role texts and the glossary are changed in their stencil and heddle files, read at call time.
- PATTERN-leaf-packages: `internal/stopclass` (new) imports only the standard library and does no I/O.
- PATTERN-tolerant-records: the stop record's new facts and the dobby's chase and post records refuse no unknown key and are written through `internal/state`.
- This task builds on `push-retry-not-halt`, which changes `internal/sheddriver/driverules.go`, the dobby's run watch and the halt classifier, and adds the driver's wait record (its D6), which D5 reads.

## Auto-mode assumptions

- Under `true` the severe classes stay as today's role text lists them, scope change and cancel included; only `unattended` narrows them.
- A severe-halt post mentions the operator under both levels, since both levels relay severe halts to Slack today.
- The fixer report's `deferred` disposition is what "a finding the fixer deferred" means.
- `beam.halt_chase_min` defaults to 10 minutes; the study shows 65 of 73 questions answered within two minutes.
- The bounds in the table are the study's proposed ones, with lym's board-claim bound one, so that every row reads lym, then orch, then severe.

## Open risks

## Acceptance criteria

- D4: a test pins that the dobby resumes a run handed back for a pause once that pause lifts, matched by the pause entry's scope,
  and leaves alone a run another pause covers, a run that moved on, and a run whose pause the orch set as a hold,
  whose entry stands through a `lym tender resume` with no slug and is lifted by one naming the slug;
  that a hold over the operator's slug pause sets its hold reason on that entry, and a hold over a bolt-hold entry is replaced by a hold entry when the dobby lifts the bolt-hold, neither resuming the run;
  and that the lift resume of a run whose driver has not parked passes the no-wait option, does not block the tick and resumes the run on a later tick; tier1 or the lowest that fits.
