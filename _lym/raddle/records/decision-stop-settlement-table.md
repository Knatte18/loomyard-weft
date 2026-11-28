## Goal

The settlement table: stop classes and who settles each, per autonomy level and halt count

Takes: D2's table and its verdict order; nothing from another part.
Delivers: D2's table steps: the new leaf package `internal/stopclass`, standard library only and no I/O, holding one named class constant per row of D2's table and the one function that returns a verdict for a class, a level and the halt count at the row, read in D2's order. lym's bound per class lives here, and lym's column applies at every level. The driver rows that take their bound from it, and the `settle: orch` hand-back, are built by `driver-settles-stops`.
Exports: Fixed for consumers: `stopclass.Class` with one constant per row; `stopclass.Level` with `LevelOff` ("false"), `LevelOn` ("true") and `LevelUnattended` ("unattended"), the same strings `dobbyengine.AutonomyLevel` holds, so a consumer converts with `stopclass.Level(level)`; `stopclass.Verdict` with `VerdictLym`, `VerdictOrch`, `VerdictOperator`; `stopclass.Settle(class Class, level Level, haltsAtRow, lymSettles int) Decision`, `Decision{Verdict, Bound}`.

## Scope

The part changes these packages:

- internal/stopclass (about 480 changed lines)

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

- D2: a tier0 test in `internal/stopclass` (new) pins the table's verdict for every class at each level, `false` among them, and at halt counts one, two and three, read in D2's order,
  that no class reaches `operator` at `false`, and that the package imports only the standard library.
