## Goal

The arbiter takes budget escalations, and nothing lands a round with an open blocking finding unattended

Takes: D6's Go and arbiter-stencil steps: `Eligibility` and `Stop` in `internal/arbiterstop/arbiterstop.go`, built in `arbiterstop.Check` from `NewReaders`; the arbiter call in `internal/shedarbiter`; the stencil `contracts/stencils/loom/loom-template-arbiter.md`; the Bouncer's escalation and accept paths in `internal/shedadapters/bouncer.go` and `readCirclingFile` and `recordAutoContinue` in `internal/shedadapters/circling.go`; `circlingVerb` and `finalizeCirclingVerb` in `internal/loomcli/circling.go`; the fixer report's `deferred` disposition parsed by `internal/burlerengine/fixerreport.go`; `dobbyengine.LookupAutonomy` from autonomy-unattended-level.
Delivers: Cause `budget` admitted; `arbiterstop.Stop` carrying the run's level, named in the arbiter's prompt; the arbiter's `budget` rules in D6's order, with the one continue per segment read from the circling decision files and refused at the hard cap; one board note for a deferred-findings accept at Code-Review and after, filed by the driver through a seam the loomcli driver wiring backs with `internal/boardengine`, with no board grant for the arbiter; under `unattended` the Bouncer runs every perch as with `blocking_escalates`, and neither the arbiter at any cause nor a Finalize fix-review accept lands a round with an open BLOCKING finding or a red gate. The orch's side of D6, the hold and the role text, is built by orch-unattended-rules.
Exports: The `Autonomy` field on `arbiterstop.Stop`.

## Scope

The part changes these packages:

- internal/arbiterstop (about 160 changed lines)
- internal/shedarbiter (about 90 changed lines)
- internal/shedadapters (about 230 changed lines)
- internal/sheddriver (about 160 changed lines)
- internal/loomcli (about 150 changed lines)
- contracts/stencils/loom (about 60 changed lines)

## Decisions

### D6 — The arbiter takes `budget` escalations; open blocking findings never land unattended

`arbiterstop.Eligibility` admits cause `budget` beside `hard-cap`, `circling`, `decision` and `stalemate`.
`arbiterstop.Stop` gains the run's autonomy level, read through `dobbyengine`'s lookup where the stop is built, and the arbiter's prompt names it.

Under `unattended` the arbiter never accepts a round that holds an open BLOCKING finding or a red live gate, whatever the escalation's cause:
where it would accept such a round, it hands the escalation on to the run's parent.
A `hard-cap` escalation, where `lym loom circling continue` refuses, is then always handed on.
Under `unattended` the Bouncer also runs every perch as with `blocking_escalates` set,
so neither its auto-accept nor its soft-cap rule's accept passes a round whose ledger holds an open BLOCKING entry (`internal/shedadapters/bouncer.go`).

The Bouncer escalates `budget` on an open BLOCKING ledger entry, on a red live gate,
and on an input it could not read: the live state, or the round's review, ledger, previous decision or previous ledger
(`settleUndecided`, `softCapRule`, `blockingEscalationDetail` and `liveRedDetail` in `internal/shedadapters/bouncer.go`).
An unreadable escalation record ends the round as a Stuck instead, and no escalation reaches the arbiter.
The arbiter settles a `budget` escalation by these rules, in order:

- An escalation whose detail names an input that could not be read, one with no cause detail, which the Bouncer raises only over an input it could not read,
  or one whose round's review, ledger or live state the arbiter cannot read itself,
  is handed on at every level, with neither accept nor continue, as the arbiter's stencil rules today.
- With the live gate green and every open BLOCKING finding one the last fixer report marks `deferred`, it accepts the round under `true`;
  at a row past the task's scope, Code-Review and after, the driver, acting on that accept, files one board note for the run listing those findings,
  through `internal/boardengine`, and the arbiter's settings gain no board grant.
  Under `false` and `unattended` it hands the escalation on.
- A missing fixer report, or one older than the round, marks no finding `deferred`.
- Otherwise it buys one more round with `lym loom circling continue`, once per review segment;
  at or past the perch's hard cap, where `circlingVerb` in `internal/loomcli/circling.go` refuses the continue, it hands the escalation on in place of the continue.
  It reads that it continued from the segment's circling decision files, `round-<n>-circling-decision.md`:
  one with decision `continue`, cause `budget` and the automatic flag false, the flag `readCirclingFile` in `internal/shedadapters/circling.go` reads, means the segment's continue is spent;
  the Bouncer's soft-cap continue, which `recordAutoContinue` writes with the flag true, spends nothing.
  This per-segment count, not D3's count over the row's history, bounds the continue.
- When an open BLOCKING finding or a red gate remains after that round, it hands the escalation on to the run's parent.

Under `true` the orch then accepts or continues, as today.
Under `unattended` the orch never accepts a round with an open BLOCKING finding, deferred or not, or a red gate, nor one the arbiter handed on over an input it could not read:
it pauses the run as a hold (D4) with a note naming the findings, or the unread input and the escalation's way forward, and posts a quiet `[operator] held <slug>:` line (D8), and other runs go on.
The same holds at a run halted at Finalize after its fix reviews ran out of rounds, which `lym loom circling accept` settles through `finalizeCirclingVerb` in `internal/loomcli/circling.go`
by landing the fixer's change with its open findings appended:
under `unattended` neither the arbiter nor the orch runs that accept while an open BLOCKING finding or a red gate remains, and the orch holds the run as above.

Bound: the arbiter accepts a `budget` round only under `true`, only green, only with every input read and only when every open BLOCKING finding was deferred;
it continues a `budget` round at most once per segment, and never at or past the hard cap;
under `unattended` nothing lands a round with an open BLOCKING finding or a red gate, neither the Bouncer, nor the arbiter at any cause, nor the orch,
and neither does a Finalize fix-review accept.
At `false` D6 changes only that the arbiter takes a `budget` escalation and may continue it once; it accepts none, and one it hands on reaches the operator as today.

Rationale: open point 3 of the study: "accept with a note" for open blocking findings lands known defects unattended, which a landing makes hard to undo;
holding that one run costs only its own time while other work continues.

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

- D6: a tier0 test in `internal/arbiterstop` pins that cause `budget` is admitted and that `arbiterstop.Stop` carries the run's autonomy level.
- D6: a test pins the arbiter's rules: a green `budget` round whose open BLOCKING findings are all `deferred` is accepted under `true`, with one board note at Code-Review and after,
  and handed on under `false` and `unattended`; a missing or older-than-the-round fixer report marks no finding `deferred`;
  a round with a red gate or an open BLOCKING finding not deferred is continued once per segment, read from the segment's circling decision files, and then handed on;
  a segment the Bouncer auto-continued still gets the arbiter's one continue, and a `budget` escalation at the hard cap is handed on, not continued;
  a `budget` escalation over an unreadable ledger with a green gate under `true` is handed on, neither accepted nor continued;
  a `hard-cap` escalation holding an open BLOCKING finding under `unattended` is handed on, not accepted;
  a run halted at Finalize after its fix reviews ran out of rounds with an open BLOCKING finding under `unattended` is held, not accepted;
  and under `unattended` the Bouncer's auto-accept and soft-cap accept pass no round with an open BLOCKING entry; tier1 or the lowest that fits.
