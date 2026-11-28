## Goal

The orch role texts gain the unattended level's rules and the glossary names the level

Takes: D9 and D13 whole and the role-text steps of D6 and D7, written in `contracts/orchrole/role-autonomy.md`, `contracts/orchrole/orch-role.md` and `internal/heddleengine/glossary.md`; the hold reason from dobby-pause-lift-resume, the arbiter's hand-on from arbiter-budget-escalations, the `settle: operator` post from severe-halt-slack-post and the stall posts from dobby-stall-chase, each landed. D9's autonomy line in `internal/orchengine/prompt.go` is built by autonomy-unattended-level.
Delivers: The unattended rules D9 lists; D6's hold of a round with an open BLOCKING finding or a red gate and of a Finalize halt while one remains; D7's relay rule, keyed posts, the in-pane relay while the slack button is off, the hold after a severe relay and the `lym selfreport create` filing, with "lym classifies no halt" dropped; and D13's three glossary lines.
Exports: Nothing in Go.

## Scope

The part changes these packages:

- contracts/orchrole (about 200 changed lines)
- internal/heddleengine (about 10 changed lines)

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

### D7 — A halt carries its class, verdict and count, and the dobby posts each severe halt to Slack

Every halt the driver hands back records four facts beside the level:
`class:`, the table's class; `settle:`, the verdict, `orch` or `operator`; `halts_at_row:`, the count (D3);
and `halt:`, the halt's key, its row and its stop identity (D3).
So every stop gets its own key, a halt that folds into the same entry and a second failed halt at the row among them.
A halt that leaves a history entry records the four on that entry in the child status.
A halt that leaves none, a producer error that leaves the run `failed`, records them on the driver's stop record (D3),
never on the newest entry, which belongs to an earlier verdict.
The dobby's child read, `ChildReading`, which today takes no field of a history entry, gains the four from where the halt recorded them:
the stop record names the open stop's halt entry, when it has one, and the child read reads the four from that entry, or from the stop record when it names none.
The next transition's entry, a fold that raises the entry's `repeats`, or the run's next stop supersedes them.
The hand-back notice to the orch carries the same four stamps beside `autonomy:` when an orch strand is recorded, as today.

The dobby that drives the run posts each halt whose verdict is `operator`, when the run's level is `true` or `unattended`,
to `<shortname>:SLACK` under its own program name, so the relay routes it to the dobby's orch, the orch of the run's worktree.
The post is one line opening `[operator] severe halt <slug> <key>:`, with the halt's `halt:` key, then the halt, what lym tried and the one action it asks for.
The dobby's state record keeps that the halt was posted, by its key, written after the post succeeds;
a dobby restarted between the post and the write posts it once more, which is accepted over losing it.
The post does not depend on a recorded orch strand.
With autonomy `false` nothing is posted to Slack by lym, and the orch relays as today.

The orch relays to Slack only the severe halts it classifies itself, the rows D2 marks *(orch)*:
under `true` a scope change or a cancel, under `unattended` a run that cannot finish within its record, and at either level a step only the operator can take.
It relays a halt with no settling action open under `true`;
under `unattended` it leaves that halt to D10's post, which reaches the operator at most two chase intervals after the halt, so one halt gets one severe post.
Under `unattended`, once the orch relays a halt it classifies as severe, it holds the run as a hold (D4), which ends D10's chase;
an agent question that shows a run cannot finish within its record is relayed as that run's severe halt, held the same way,
and answered that the run is held, which ends that question's chase.
Its relay opens `[operator] severe halt <slug> <key>:` with the `halt:` key its notice carries;
a halt it classifies with no such notice, a scope change or a cancel, carries no key.
It never relays a halt whose notice reads `settle: operator`, which lym posted,
except while the dobby's orch has its slack button off (D12), when it relays in its pane such a halt, as today, and each stall D10 posts as severe.
For a `settle: operator` halt that is a third halt at a row, the orch still files the bug with `lym selfreport create`, without relaying it.
The orch's role texts, `role-autonomy.md` and the orch role, say so, and drop "lym classifies no halt".

Bound: lym posts to Slack only a halt the table makes severe, or one D10 escalates, and only under autonomy; every other halt reaches only the orch, as today.

Rationale: a severe halt must reach the operator even when the orch is slow or gone overnight,
and the dobby already posts its wait reminders to the hub's Slack program as a program.

### D9 — Under `unattended` the orch never asks the operator

`contracts/orchrole/role-autonomy.md` gains the unattended level's rules, read when a notice carries `autonomy: unattended`:

- The severe classes narrow: a scope change and a cancel are no longer severe.
  A scope change is filed as a board note and the run finishes within its decision record;
  only a run that cannot finish within its record is severe.
- The orch never runs `lym batten cancel`: a run it would cancel, an entry overtaken by landed work among them,
  is paused as a hold with a note saying why and posted as a quiet `[operator] held <slug>:` line.
- A design call or circling stop the arbiter hands on is settled with the narrower reading, recorded with `lym loom decision add`.
- An agent's question is answered and never relayed, but for one that shows a run cannot finish within its record, which the orch relays as that run's severe halt and holds (D7).
- A defect or hub config stop it settles is also filed with `lym selfreport create`.
- A review round, and a run halted at Finalize after its fix reviews ran out of rounds, is held or accepted as D6 says.

`internal/orchengine/prompt.go`'s autonomy line names the level, so the orch reads which rules apply.

Rationale: the operator decided that under the stricter level the orch never asks the operator and only severe halts go to Slack;
the rules are the "Overnight" column of the study's table.

### D13 — The glossary names the level and who classifies a severe halt

`internal/heddleengine/glossary.md` gains and changes these lines:

- **unattended autonomy**: autonomy's stricter level, `beam.autonomy: unattended`, in which the orch never asks the operator and only a severe halt reaches them, on Slack. Never: overnight mode, full auto, autopilot.
- **severe halt**: a halt in one of autonomy's closed classes, which the settlement table or the orch classifies and which reaches the operator. Never: critical halt, blocker.
- **settlement table**: the one table in lym that decides, per stop class and autonomy level, whether lym, the orch or the operator settles a stop. Never: rule book, policy.

Rationale: the operator left the level's name open.

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

- D6, Review: `role-autonomy.md` says the unattended orch pauses as a hold, never accepts, a round holding an open BLOCKING finding or a red gate,
  and a run halted at Finalize after its fix reviews ran out of rounds while one remains.
- D7, Review: the orch role texts say the orch relays only the severe halts of the rows D2 marks *(orch)*, a cancel under `true` among them, and leaves a halt with no settling action open to lym's post under `unattended`;
  that it never relays a halt whose notice reads `settle: operator` but in its pane while the dobby's orch has its slack button off, where it also relays each stall D10 posts as severe;
  that under `unattended` it holds a run whose halt it relayed as severe, and relays a question that shows a run cannot finish within its record as that run's held severe halt;
  that it still files the bug of a `settle: operator` third halt at a row with `lym selfreport create`; and they no longer say lym classifies no halt.
- D9, Review: `role-autonomy.md` holds the unattended rules D9 lists, the narrowed severe classes among them, and leaves the `true` rules unchanged but for D7's relay rule.
- D13, Review: `internal/heddleengine/glossary.md` holds the unattended autonomy, severe halt and settlement table lines as D13 gives them.
