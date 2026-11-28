## Goal

beam.autonomy takes false, true or unattended, and every reader reads the level

Takes: D1 whole: `LookupAutonomy` in `internal/dobbyengine/tasksetting.go`, `checkTaskSetting` in `internal/dobbyengine/config.go` and its callers, the template comment in `internal/dobbyengine/template.yaml`, `hubgeom.Autonomy` in `internal/hubgeom/autonomy.go` and its callers in slackcli, loomcli, orchcli and beamcli, the beam modes in `internal/boardengine/beammodes.go`, and the autonomy line in `internal/orchengine/prompt.go`, which also serves D9's rule that the line names the level.
Delivers: The three levels on every path that checks the key, a refusal naming the three values and `lym config beam`, a refused value read as `false` and posted once per episode to the orch, the level on the `autonomy:` stamp, `off`, `on` or `unattended` in the beam modes, and the level named in the orch prompt and the template comment. Every reader that asks only whether autonomy is on reads `true` and `unattended` as on.
Exports: Fixed for consumers: `dobbyengine.AutonomyLevel`, a string type with `AutonomyOff` ("false"), `AutonomyOn` ("true") and `AutonomyUnattended` ("unattended") and an `On() bool` method; `dobbyengine.LookupAutonomy` and `hubgeom.Autonomy` return it.

## Scope

The part changes these packages:

- internal/dobbyengine (about 260 changed lines)
- internal/hubgeom (about 70 changed lines)
- internal/boardengine (about 40 changed lines)
- internal/orchengine (about 30 changed lines)
- internal/orchcli (about 40 changed lines)
- internal/slackcli (about 15 changed lines)
- internal/loomcli (about 30 changed lines)
- internal/beamcli (about 45 changed lines)

## Decisions

### D1 — `beam.autonomy` takes `false`, `true` or `unattended`

`beam.autonomy` stays one per-task key, read only through `dobbyengine`'s one lookup, and takes three values:

- `false`: no autonomy: the orch relays to the operator every halt it does not settle, as today.
- `true`: autonomy as today's `contracts/orchrole/role-autonomy.md` defines it.
- `unattended`: unattended autonomy, the stricter level in which the orch never asks the operator and only a severe halt reaches them (D9).

The values `false` and `true` keep their meaning, so no `configreg.Reinterpretation` ships, and a board `run_settings` holding a boolean stays valid.
`dobbyengine`'s one value check for per-task settings learns the third value, so every path that checks the key accepts it alike:
the lookup, the beam config load's check of the resolved file, `lym config beam --set` and a task's `run_settings`.
Any other value is refused on each of those paths, naming the three values and `lym config beam` as the way forward.
`hubgeom.Autonomy` returns the level instead of a boolean, and every reader that asks only whether autonomy is on reads both `true` and `unattended` as on.
A value the lookup refuses reads as `false` there, the level at which the orch asks the operator,
and the dobby posts the refusal once to its orch per episode, so a mistyped level does not pass unseen.
A halt notice's `autonomy:` stamp carries the level, so the orch handles a halt under the level its notice names, as today.
The board's beam modes show `off`, `on` or `unattended`.
The orch prompt's autonomy line names the level, and the template comment on `beam.autonomy` names all three values.

Bound: a binary older than this change refuses the beam config while a layer holds `unattended`, as it refuses any value it does not know;
the operator sets `unattended` only once every machine of the hub runs a build that knows it, and the template comment says so.

Rationale: the operator decided that the overnight mode belongs in autonomy itself, as a second, stricter level beside today's.
Keeping the booleans' meaning keeps every hub's settings valid without a reinterpretation, as PATTERN-config-layers requires only for a meaning change.

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

- D1: a tier0 test in `internal/dobbyengine` pins that the lookup, the beam config load's check and the local values check accept `false`, `true` and `unattended`,
  that a boolean in a board `run_settings` still reads, and that any other value is refused naming the three values and `lym config beam`.
- D1: a test pins that `hubgeom.Autonomy` returns the level and reads a refused value as `false`, that a halt notice's `autonomy:` stamp carries the level,
  that the board's beam modes show `off`, `on` or `unattended`, and that the dobby posts a refused value once per episode; tier1 or the lowest that fits.
- D1, Review: the template comment on `beam.autonomy` names the three values and says to set `unattended` only once every machine runs a build that knows it, and the orch prompt's autonomy line names the level.
