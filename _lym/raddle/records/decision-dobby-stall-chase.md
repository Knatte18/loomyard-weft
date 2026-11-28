## Goal

The dobby chases stalls the orch leaves unsettled, reminds of severe posts, and warns when autonomy cannot reach the operator

Takes: D10 and D12 whole and D8's reminder steps: severe-halt-slack-post's severe-post record and post; `creelcli.OpenQuestionProbe` from idle-notice-correct-waits; the respawn cap the child read shows; the dobby `ConfigSpec` in `internal/dobbyengine/config.go`; the slack button state and the configured operator, read through the dobby's wiring in `internal/beamcli/dobbydeps.go`.
Delivers: D10: `beam.halt_chase_min`, default 10, classified `CategoryNone` and refused under 1 naming the key in full; one chase line per stall key to the dobby's orch for a `settle: orch` halt, an open creel question and a dead driver past its caps, naming `lym loom resume` when a parent decision came after the halt; under `unattended` one severe post per stall key one interval later, also sent to the orch while its slack button is off; chase and post kept in the state record and ended by D10's end events. D8: the wait reminders remind of each severe post the state record holds at `slack.remind_after_min`, without a mention, until its end event. D12: one warning per episode when a task resolves `true` or `unattended` with the slack button off or no operator configured, naming `lym slack`.
Exports: The `beam.halt_chase_min` key.

## Scope

The part changes these packages:

- internal/dobbyengine (about 620 changed lines)
- internal/beamcli (about 90 changed lines)
- internal/battencli (about 40 changed lines)

## Decisions

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

### D10 — The dobby chases a stall the orch leaves unsettled under autonomy

Under `true` or `unattended`, the dobby chases three stalls after `beam.halt_chase_min` (new) minutes, 10 by default, with one chase line to the dobby's orch naming the stall and its key:

- a run whose halt reads `settle: orch` (D7), on its history entry or on the driver's stop record, and that is still halted under that halt's `halt:` key;
- an agent's open question in creel, a message of its own that expects a reply, neither answered nor settled (D11), keyed `question-<id>` by its creel id;
- a dead driver past the respawn and revive caps the dobby and batten keep today, which the child read's respawn cap shows,
  keyed `driver-<row>-<time>` by its row and the time the dobby first read it dead.

When a parent decision was recorded with `lym loom decision add` after the halt and the run was not resumed, the chase says so and names `lym loom resume` as the way forward.
Under `unattended`, a stall still unsettled one interval after the chase is posted as a severe halt with the stall's key, as D7 posts one, and D7's state record and D8's reminders cover it.
The relay drops that post while the dobby's orch has its slack button off (`postMessages` in `internal/slackengine/outbound.go`),
so the dobby then also sends its orch the post's line, and the orch relays the stall in its pane, as it relays a `settle: operator` halt (D7).
These end the chase: for a halt, any transition of the run, a resume, a goto or an accept among them, or a pause covering the run, the orch's hold among them;
for a question, its answer or its settle; for a dead driver, its revive or a transition of the run.
The dobby's state record keeps the chase and the post, so a restarted dobby sends neither again.
The key is classified `CategoryNone` in the dobby's `ConfigSpec`, and a value under 1 is refused naming the key, as PATTERN-config-key-names requires.

Bound: one chase per stall key, and under `unattended` one severe post per stall key after it; a run at a wait by design, or held by a pause, is never chased.

Rationale: the study's recurring operator pushes were a halt the orch had answered without releasing the run and a question nobody acted on;
overnight an orch that stalls must not leave a run halted until morning.

### D12 — The dobby warns once when autonomy cannot reach the operator

When a task resolves `beam.autonomy` to `true` or `unattended` and the dobby's orch has its slack button off, or the relay has no configured operator to mention,
the dobby posts that orch one line saying severe halts of the task, and the stalls D10 posts as severe, would reach no one on Slack, naming `lym slack` as the way forward.
It posts once per episode and again after the cause cleared and returned.
While the button is off the orch relays in its pane a `settle: operator` halt and each stall D10 posts as severe (D7, D10).

Bound: the warning changes nothing and switches nothing; the relay still drops mail of an orch whose button is off, as today.

Rationale: the relay drops the mail of an orch whose button is off and can mention only a configured operator,
so a night under autonomy without either would hide every severe halt lym posts.

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

- D8: a tier0 test in `internal/dobbyengine` with an injected clock pins that the wait reminders remind of each severe post the state record holds at `slack.remind_after_min` while the operator is away,
  and stop at the post's own end event: once the run leaves the halt, once a question stall's question is answered or settled, and once a dead-driver stall's driver is revived.
- D10: a tier0 test in `internal/dobbyengine` with an injected clock pins one chase after `beam.halt_chase_min` for a run still halted with `settle: orch`,
  for an agent's open question in creel, and for a dead driver past its caps,
  the chase naming `lym loom resume` when a parent decision was recorded after the halt, no chase after a transition, an answered question, a revive or once a pause covers the run,
  under `unattended` one severe post with the stall's key one interval after the chase and none for a paused run, and neither sent again by a restarted dobby;
  that the chase line carries the stall's key, and that with the dobby's orch's slack button off the post's line is also sent to that orch;
  and that a halt the orch relayed as severe under `unattended` and then held gets no second severe post.
- D10: a tier0 test pins that `beam.halt_chase_min` under 1 is refused naming the key in full.
- D12: a tier0 test in `internal/dobbyengine` pins one warning when a task resolves `true` or `unattended` with the dobby's orch's slack button off or no operator configured,
  none again in the same episode, and one after the cause cleared and returned.
