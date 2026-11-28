## Goal

The driver settles board claim refusals, failed launches, bounce budgets and gate attempts itself within the table's bound

Takes: D4's driver rows, D2's driver steps and D3's bound check, over stop-halt-count's count function and stop record and stop-settlement-table's bounds. Raise sites: the claim push in `pushOwnerCommit` in `internal/boardengine/ownerset.go`, reached from `claimBoardEntry` in `internal/battencli/owner.go`; `shuttleengine.ErrNotStarted` and `shedtransient.TransientAgentStart`; the session timeout reason in `internal/shedadapters/multillm.go`; the fixer fork in `internal/websterengine`; `strandGoneError` in `internal/shuttleengine/run.go`; and the gate-attempts Stuck from `gateFailedReason` in `internal/shedadapters/gateattempts.go`.
Delivers: New rows in `decideDrive` in `internal/sheddriver/driverules.go`: a board claim refusal gets a board sync and a re-step; a launch never ready, a session timeout, a failed fixer fork and a lost strand get one fresh start of the row's producer; a bounce budget exhausted gets one resume, matching the `shedengine.ReasonBounceBudgetExhausted` prefix; a gate-attempts Stuck gets a re-step with a fresh producer session. Each class but the bounce budget's is raised as a named kind or field where the stop starts. Each row takes its bound from the table, a settle within the bound sends no notice and leaves a settle record, and a stop past it is handed back with its verdict. The tackler keeps every stop no row names.
Exports: The named kinds for the claim refusal, the session timeout, the fixer fork failure, the lost strand and the gate-attempts Stuck.

## Scope

The part changes these packages:

- internal/sheddriver (about 650 changed lines)
- internal/boardengine (about 70 changed lines)
- internal/battencli (about 30 changed lines)
- internal/shuttleengine (about 130 changed lines)
- internal/shedadapters (about 140 changed lines)
- internal/shedtransient (about 50 changed lines)
- internal/websterengine (about 80 changed lines)

## Decisions

### D2 — One settlement table decides who settles each stop class

A new leaf package, `internal/stopclass` (new), holds the stop classes as named constants and the one settlement table.
For a class, an autonomy level and the halt count at the row (D3) it returns a verdict: `lym`, `orch` or `operator`, with lym's bound for the class.
The package imports only the standard library and does no I/O, so the driver, the dobby and tests read the same table.
No other Go package decides who settles a stop class; the driver's and the dobby's rules for a class lym settles take their bound from the table.

The verdict at a stop is read in this order:

1. `operator` when the level is `true` or `unattended` and the row's operator column names the case, the third halt at the row among them.
2. `lym` while lym's settles of the class at the row are within lym's bound.
3. `orch` otherwise.

At `false` the operator column never applies, so the table returns `lym` within lym's bound and `orch` otherwise,
and the orch relays to the operator every halt it does not settle, as today.
lym's column applies at every level: D4's rows settle at `false` too, and D6 states what the arbiter's rules change at `false`.

| Class | lym settles | Orch, `true` | Orch, `unattended` | Operator (severe), `true` and `unattended` |
|---|---|---|---|---|
| Board claim push refused at `Worktree-Create`, a start failing on a held board lock among them | board sync, then re-step; one | settles | settles | third halt at the row |
| Pause covers the run | the dobby resumes it once the pause lifts (D4) | — | — | — |
| Agent launch failed, session timed out, fixer fork failed, strand lost | one fresh start | settles | settles | third halt at the row |
| Stale build | hand-over to a fresh build, as today | when no newer build exists | when no newer build exists | third halt at the row |
| Escalation `hard-cap`, `circling`, `decision`, `stalemate` | the arbiter, as today; under `unattended` it hands on a round holding an open BLOCKING finding or a red gate (D6) | what the arbiter hands on | what the arbiter hands on (D6, D9) | third halt at the row |
| Escalation `budget` | the arbiter (D6) | what the arbiter hands on | what the arbiter hands on (D6) | third halt at the row |
| Bounce budget exhausted | one resume | settles | settles | third halt at the row |
| Gate attempts exhausted | one re-step with a fresh producer session | settles | settles | third halt at the row |
| Webster failure, run-exit audit finding, defect or hub config at a row | — | settles | settles, and files the bug with `lym selfreport create` | third halt at the row |
| Landing stop, deploy failure at `Land` | as today | settles | settles | third halt at the row |
| `driver_not_parked` | the verb waits (D5) | — | — | — |
| Dead driver, agent API error | respawn or revive within the respawn and revive caps the dobby and batten keep today, which this row records and does not change | after the cap | after the cap | `unattended`: still dead one interval after D10's chase |
| Agent question, entry overtaken, scope change, cancel *(orch)* | — | as today's role says | answers, never relays (D9) | `true`: a scope change or a cancel; `unattended`: a run that cannot finish within its record |
| Owner halt, takeover | — | — | — | always |
| Step only the operator can take, no settling action open *(orch)* | — | — | — | always |
| Unnamed stop | the tackler, as today | what the tackler hands on | what the tackler hands on | third halt at the row |

Two rows are no stop at a row, so no halt count reaches their operator column:
a `driver_not_parked` refusal answers a verb and leaves the run as it was, and a dead driver or an agent API error is reported by the dobby's notices (D3).
Neither can repeat at a row unseen: the verb waits instead of refusing (D5), and a driver dead past its cap reaches the orch and D10's chase.

The rows marked *(orch)* hold classes the driver cannot detect:
an agent question, an entry overtaken, a scope change, a cancel, a step only the operator can take and a halt with no settling action open.
The table lists them for the role text, lym stamps none of them on a stop, and the orch classifies them from what it reads (D7).

A raddle split's approval, a merge-ready beam and a run at `PR-Review` are waits by design, not stops, and are not in the table (D8).
A stop lym settles within its bound sends the orch no notice; the driver's settle record keeps it.
Every other stop is handed back with its verdict, `orch` or `operator` (D7), so every hand-back to the orch carries `settle: orch`.

Rationale: the operator asked for a rule table that decides per stop class who settles it, with the severe classes the only ones that reach the operator.
A table in a leaf package is the one place that decision lives, so the driver's bounds, the dobby and the orch read one answer.

### D3 — The halt count at a row counts lym's own retries

The halt count at a row is the number of stops of the run at that row, each counted once.

A stop does not always leave a history entry:
a producer that returns an error returns no outcome, `appendHistory` in `internal/shedengine/run.go` then appends nothing, and the run is persisted `failed`.
An agent launch that never became ready and an agent that died before its verdict stop this way.
The stop key, the row, the history length and the state, is then the same for two such stops at one row, so it cannot tell them apart.
The driver therefore gives each stop it steps a stop identity, kept in a stop record it writes through `internal/state` beside its settle records.
Before it calls a step, the driver closes the record's open stop.
When the step ends in a stop, it opens a new identity there.
The open stop holds its stop key, the history index of its halt entry when it has one, and the four facts D7 records once the halt is handed back.
A restarted driver that reads a stopped run whose stop key matches the record's open stop takes over that identity, so a driver restart opens none;
one that finds the record's stop closed opens a new identity, so a driver that died between the step's write and the record's counts the stop all the same.
Every settle record of a stop carries the stop's identity, the hand-back and arbiter `restep` records among them,
beside the stop's class and the history index of the halt entry it settled, when there is one.

Each stop counts once, through what it leaves:

- A halt entry in the status history, as today's class-5 rule counts it: a Stuck at the row with no `routed_to`, plus its `repeats`.
- The Awaiting entry a review escalation leaves when the arbiter hands it on, which counts once;
  the class-5 rule does not count Awaiting entries today, and this is the one it gains.
- A stop that leaves no history entry, through its stop identity, whoever settled it: lym's fresh start, a hand-back the orch resumes, or a tackler escalation the arbiter resumes.

A settle record adds no stop of its own: it counts with the entry or identity of the stop it names as one stop.
So a retry of lym's, a re-step after a fault, a fresh start or a resume past a budget, counts through the stop it settled,
and so does a tackler resume, settler `tackler`, action `restep`, which `tackle` in `internal/sheddriver/drive.go` writes;
the tackler attempt and the tackler's `spawn` settle record count nothing.

These count nothing:

- a wait: a network wait under `push-retry-not-halt`'s rule, a pause, a `driver_not_parked` wait, a hand-over to a newer build;
- a review escalation the arbiter settles, by resume or accept, since it ends a review round as designed;
- an accept that ends a review segment, since it moves the run on;
- a dead driver or an agent API error, which the dobby reports from its notices and which is no stop at a row;
- an interrupted-reinvoke re-step after a driver wake, which carries on a step the dead driver left and opens no stop identity;
- a settle record that retried nothing, matched by its settler, its action and the stop's class it carries:
  an arbiter's `restep` or `spawn` record, a tackler's `spawn` record, a `go-rule` `restep` of an interrupted step under the reinvoke policy, and every `moved`, busy-exit, running-exit or hand-back record.
  Such a record excludes only a stop this list already excludes; a stop the list above counts still counts.

The count runs over the run's whole history at the row, as today's rule does.
It matches a settle record to its row by the row and class it carries, and to its stop by the stop identity, never by the exact stop key.
lym's bound for a class (D2) counts lym's settles of that class at the row over the same history,
so the driver's wake restart of its repair count, `wakeRestartGap`, never restores a bound the table set.
One function computes both counts, in `internal/sheddriver`, which owns the settle records and the stop record.

Bound: a class whose verdict reaches the operator at the third halt at a row does so at the third stop whoever settled the first two,
so a lym retry loop never hides a repeating stop.

Rationale: open point 1 of the study: today's text makes a repeating stop severe at the third halt at a row, and lym's own retries must count toward it.

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

- `worktree-create-transient-net` changes Worktree-Create's stop handling beside D4's board-claim row; the classes differ, a transient `ls-remote` against a refused board claim,
  and the raddle promotion names it in `depends_on` so both rows are built over one stop chain.

## Acceptance criteria

- D2: a tier0 test in `internal/sheddriver` pins that each new driver row takes its bound from the table and that a stop past the bound is handed back with `settle: orch`.
- D3: a tier0 test in `internal/sheddriver` pins that a class whose verdict is severe at the third halt reaches `settle: operator` when two earlier stops at the row were settled by lym alone,
  that a wake past `wakeRestartGap` does not restore lym's bound, that a driver restarted at a stopped run keeps the stop's identity,
  and that one that died after a step persisted a second failed stop at the row but before it opened the identity opens a new one;
  and that an agent launch failing three times at one row with no history entry, one fresh start, then a hand-back, a resume and a third failure, reaches `settle: operator`.
- D4: a tier0 test in `internal/sheddriver` pins each new row: a board claim refusal is synced and re-stepped once, then handed back;
  a launch failure, session timeout, fixer fork failure and lost strand get one fresh start, then are handed back;
  a bounce budget exhausted gets one resume, then is handed back; a gate-attempts Stuck gets one fresh re-step, then is handed back;
  and that each row matches its class by kind or field, not by message text, but the bounce budget's row, which matches the `shedengine.ReasonBounceBudgetExhausted` prefix.
- D4: a tier0 test in `internal/sheddriver` pins that a stop settled within its bound sends the orch no notice and leaves a settle record.
