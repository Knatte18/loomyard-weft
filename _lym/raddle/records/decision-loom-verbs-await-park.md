## Goal

lym loom resume and lym loom start wait for the driver to park, with a no-wait option batten passes

Takes: D5 whole: the `shedrun.StartNotParkedKind` refusals in `internal/loomcli/resume.go` and `internal/loomcli/start.go`, the driver's wait record `sheddriver.ReadDriveWait` that push-retry-not-halt added, the keep-warm cap from `shuttleengines.KeepWarm`, `childSpawnError` in `internal/battencli/wire.go` and the `battenshed.ErrChildNotParked` retry in `internal/battenshed/innerrun.go`.
Delivers: Both verbs wait for the park, or for a run lock on a non-running run to be released, under the keep-warm cap, waking at most once a second with an injected clock; answer at once for a driver that keeps a wait record; refuse past the cap naming the retry; and refuse at once with the no-wait option, which batten passes on its child start and on every resume it runs, the resume of dobby-pause-lift-resume's lift pass among them.
Exports: Fixed for consumers: the `--no-wait` flag on `lym loom resume` and `lym loom start`.

## Scope

The part changes these packages:

- internal/loomcli (about 420 changed lines)
- internal/battencli (about 60 changed lines)
- internal/battenshed (about 50 changed lines)

## Decisions

### D5 — `lym loom resume` and `lym loom start` wait for the driver to park

Where `internal/loomcli/resume.go` and `internal/loomcli/start.go` refuse with `shedrun.StartNotParkedKind` because the driver is live and not yet parked,
or its driver lock is held, the verb waits for the park and then acts.
A run lock held on a non-running run is the driver finishing its post-run work, which `resume.go` refuses today with the same kind and a retry text:
the verb waits for the lock's release under the same cap and wake rule, and then acts.
A driver that holds its driver lock and keeps a wait record, such as `push-retry-not-halt`'s network wait, is driving the run, not about to park:
the verb answers at once that the driver drives the run and what it waits on, and changes nothing, since the driver steps again once the wait ends.
The wait wakes at most once a second and backs off while idle, and since an agent blocks on it, its cap comes from the keep-warm resolver.
Past the cap the verb refuses as today, naming the retry as its way forward.
Both verbs gain a no-wait option that keeps today's refusal at once.
Batten passes it on its child `lym loom start` and on its resume, the resume of D4's lift pass among them:
`childSpawnError` in `internal/battencli/wire.go` reads the child's not-parked refusal as `battenshed.ErrChildNotParked`, which `internal/battenshed/innerrun.go` retries on a later probe,
so no dobby or batten tick blocks on the wait.

Bound: the verb waits no longer than the keep-warm cap and changes nothing while it waits; a call with the no-wait option never waits.

Rationale: the orch wrapped 16 commands in its own retry loop for this refusal, and the verb can wait for the park itself.

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

- D5: a test in `internal/loomcli` with an injected clock pins that `lym loom resume` and `lym loom start` wait for a driver that parks and then act,
  wait for a run lock held on a non-running run to be released and then act,
  answer at once and change nothing for a driver that keeps a wait record, refuse with `driver_not_parked` naming the retry once the cap passes,
  and refuse at once with the no-wait option batten's child start and resume pass; tier1 or the lowest that fits.
