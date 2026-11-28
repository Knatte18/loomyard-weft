## Goal

The driver gives each stop an identity, counts halts at a row with lym's retries, and records a halt's four facts

Takes: D3 whole and D7's recording steps; `stopclass.Settle` and the class constants from stop-settlement-table; `dobbyengine.AutonomyLevel` through `hubgeom.Autonomy` from autonomy-unattended-level, converted with `stopclass.Level(level)` in the driver wiring in `internal/loomcli/drivewiring.go`. It answers the Discussion-Review round-3 BLOCKING finding failed-state-halt-not-counted-or-recorded through the stop identity, as both added decisions rule.
Delivers: D3: the stop record written through `internal/state` beside the settle records, holding the open stop's key, its halt entry's history index and the four facts; the identity opened, closed and taken over across a driver restart as D3 says; settle records carrying the stop identity, the class and the halt entry's index; one function that computes both the halt count at a row and lym's settles of a class at the row over the whole history, counting and excluding exactly what D3 lists, so `wakeRestartGap` never restores a bound. D7's recording steps: every hand-back records `class:`, `settle:`, `halts_at_row:` and `halt:` on its history entry, or on the stop record when the stop left none, and carries them on the notice beside `autonomy:`; every hand-back to the orch reads `settle: orch` or `settle: operator`. D7's criterion, which needs the dobby's child read, is carried by severe-halt-slack-post.
Exports: Fixed for consumers: `shedengine.HistoryEntry` fields `Class`, `Settle`, `HaltsAtRow` and `Halt`; `shedrun.SettleRecord` fields `StopID`, `Class` and `HistoryIndex`; the stop record type and its path helper in `internal/shedrun`; the halt key, the row's producer and the stop identity joined, which readers treat as an opaque string; the count function in `internal/sheddriver`.

## Scope

The part changes these packages:

- internal/sheddriver (about 820 changed lines)
- internal/shedrun (about 80 changed lines)
- internal/shedengine (about 90 changed lines)
- internal/loomcli (about 40 changed lines)

## Decisions

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

- D3: a tier0 test pins the halt count at a row over a history holding a halt entry with repeats, a stop with no history entry and its settle records, a settle record naming its halt entry,
  a network wait, a pause, a build hand-over, an arbiter resume, an arbiter accept, a handed-on escalation's Awaiting entry,
  a halt entry the tackler resumed, with its tackler `spawn` and `restep` settle records and its tackler attempt, which counts once,
  and an interrupted-reinvoke re-step after a driver wake, counting each retry and the handed-on escalation once and no wait, arbiter settle, accept or wake re-step.
