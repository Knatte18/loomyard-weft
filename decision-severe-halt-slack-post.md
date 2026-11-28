## Goal

The dobby posts each severe halt to Slack, and the relay mentions the operator on a severe-halt post

Takes: D7's dobby steps and D8's relay steps: the four facts and the halt key stop-halt-count records on the history entry or the stop record; `ChildReading` in `internal/dobbyengine/runnotice.go`, read through `ReadChild` in `internal/battencli/dobbynotice.go`; the program post `internal/dobbyengine/waitreminder.go` makes, wired in `internal/beamcli/dobbydeps.go`; `internal/slackengine/outbound.go` and the operator ID in `slackengine.Setup`; `dobbyengine.AutonomyLevel` from autonomy-unattended-level.
Delivers: D7: `ChildReading` carries the four facts from the halt entry the stop record names, or from the stop record; the dobby that drives the run posts one `[operator] severe halt <slug> <key>:` line to `<shortname>:SLACK` under its program name for each `settle: operator` halt under `true` or `unattended`, whether or not an orch strand is recorded, and keeps the posted key in its state record, written after the post. D8: the relay mentions the configured operator on a post whose first line opens `[operator] severe halt <slug>`, from an orch or a program, and dedups a second one by slug and key within `slack.remind_after_min`, keyless only against keyless; every other post carries no mention. D8's reminders and D10's stall posts are built by dobby-stall-chase; the orch's relay rules by orch-unattended-rules.
Exports: The severe-post record in the dobby state record, which dobby-stall-chase's reminders and stall posts read.

## Scope

The part changes these packages:

- internal/dobbyengine (about 420 changed lines)
- internal/battencli (about 70 changed lines)
- internal/beamcli (about 60 changed lines)
- internal/slackengine (about 260 changed lines)

## Decisions

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

### D8 — A severe-halt post mentions the operator; waits by design stay quiet

The Slack relay's outbound post, in `internal/slackengine/outbound.go`, mentions the configured operator in a post whose first line opens `[operator] severe halt <slug>`,
whoever sent it, lym or an orch, so Slack notifies the operator's phone.
A second such post for the same slug and the same key within `slack.remind_after_min` carries no mention;
a post with no key dedups only against an earlier keyless post of the same slug, and a keyed post only against an earlier post of the same slug and key,
so a relay and a turn reply for one halt notify once, and a new halt of the same run, a fold into the same history entry among them, notifies again.
Every other post, `[operator] split ready`, `[operator] merge ready`, `[operator] held` and every reminder among them, carries no mention.
Slack documents that a user mention, `<@USER_ID>` in the message text, notifies that user (Slack API docs, "Formatting text for app surfaces", Mentioning users).
While the operator is away, the dobby's wait reminders, `internal/dobbyengine/waitreminder.go`, also remind of each severe post its state record holds,
at `slack.remind_after_min`, without a mention, until the end event of what it reminds of:
for a halt, the run leaving that halt; for a D10 stall, the event D10 lists that ends its chase.
A severe halt waits for the operator's answer, which reaches the orch as Slack mail; the dobby goes on starting and driving other runs meanwhile.

Bound: the mention keys on the line prefix alone, so an orch that opens a reply `[operator] severe halt` for a halt outside the classes also notifies;
the role texts permit that prefix only for a severe halt.
A post that drops a keyed halt's key mentions again, so the role texts require the `halt:` key on every post of a halt whose notice carries one.

Rationale: open point 2 of the study: a raddle approval and a pull request review are waits by design that block no other work,
so they stay quiet, and only a severe halt asks for the operator at once.

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

- `beam-member-parent` changes which orch a beam member's run reports to; D7's post goes to the orch of the dobby that drives the run, and that task decides whether that orch stays the run's parent.

## Acceptance criteria

- D7: a test pins that a hand-back records `class:`, `settle:`, `halts_at_row:` and `halt:` on the halt's history entry the dobby's child read reads,
  that a failed-state hand-back with no history entry records the four on the stop record the child read reads,
  that the next transition's entry or a fold that raises `repeats` supersedes them and gives a new key,
  that a second failed-state hand-back at the row gets a new key and a second severe post when its verdict is `operator`,
  that the dobby posts one `[operator] severe halt <slug> <key>:` line to `<shortname>:SLACK` under its program name for a severe verdict under `true` or `unattended`, with or without a recorded orch strand,
  that a restarted dobby does not post it again once its state record holds it, and that under `false` nothing is posted to Slack; tier1 or the lowest that fits.
- D8: a tier0 test in `internal/slackengine` pins that an outbound post whose first line opens `[operator] severe halt <slug>` mentions the configured operator, from an orch and from a program,
  that a second one for the same slug and key within `slack.remind_after_min` does not, that a keyless one dedups only against an earlier keyless one of the slug,
  that a keyless one following a keyed one of the same slug within the window mentions the operator,
  that one for a distinct second halt of the slug within that window does, a fold into the same history entry and a second failed halt at the row with no history entry among them,
  and that `[operator] split ready`, `[operator] merge ready`, `[operator] held` and a reminder carry no mention.

## PATTERN changes

- `PATTERN-stop-settlement` — adds: Deciding who settles a run stop: only `stopclass`'s one table, per stop class and autonomy level; a lym retry counts toward the row's halt count; lym posts only a severe verdict to Slack.
