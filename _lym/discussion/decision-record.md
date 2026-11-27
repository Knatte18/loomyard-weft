## Goal

lym runs overnight with no one to answer questions.
Today runs stop on known classes of stop that a Go program could settle, and the operator has to tell the orch that a run stands still.
After this task one settlement table in Go decides, for each stop class and autonomy level, whether lym, the orch or the operator settles it.
lym settles the mechanical classes itself within a bound and reports only what it could not settle.
A new, stricter autonomy level, unattended autonomy, lets the orch settle everything that is not truly severe without the operator.
A severe halt waits for the operator's answer and reaches them on Slack as a message that notifies their phone, while other work goes on.
An idle notice no longer fires for an agent that waits correctly; a notice that does fire names what the agent waits on.

The design rests on `docs/research/autonomy/stop-classes-2026-11-27.md` and settles the three open points of its section 3 (D3, D6, D8).

## Scope

In:

- The levels of `beam.autonomy` and the unattended level's rules for the orch.
- The settlement table, its stop classes, the halt count at a row, and the verdict a halt carries.
- The driver's and the dobby's settling of the mechanical classes the table gives lym, beyond the classes `push-retry-not-halt` settles.
- `lym loom resume` and `lym loom start` waiting for the driver to park.
- The arbiter taking review escalations of cause `budget`.
- The dobby's Slack post of each severe halt, the operator mention, and reminders of a severe halt.
- A chase of a halt the orch leaves unsettled under autonomy.
- The idle notice's reading of correct waits and its naming of what an agent waits on, and the noisy notices section 5 of the study names.
- The orch role texts, the config comments and the glossary lines the levels change.

Out:

- The network, GitHub API, push lock, failed-push, catch-up and landing network classes:
  `push-retry-not-halt` settles them, by its own rules and bounds, and they are not rows of this task's table.
  A refusal that task stops at once and only the operator can clear, such as refused credentials, is the class "step only the operator can take" (D2).
- Worktree-Create's transient `ls-remote` failure, which `worktree-create-transient-net` owns.
- The parent of a beam member's run, which `beam-member-parent` owns.
- The classes the study's table "Classes that stopped recurring" lists; a recurrence of one reaches the tackler as an unnamed stop does today.
- A lym runner for a measurement an agent may not run, and lym answering a provider permission dialog (support log, Rejected alternatives).

## Decisions

### D1 — `beam.autonomy` takes `false`, `true` or `unattended`

`beam.autonomy` stays one per-task key, read only through `dobbyengine`'s one lookup, and takes three values:

- `false`: no autonomy, as today.
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

### D2 — One settlement table decides who settles each stop class

A new leaf package, `internal/stopclass` (new), holds the stop classes as named constants and the one settlement table.
For a class, an autonomy level and the halt count at the row (D3) it returns a verdict: `lym`, `orch` or `operator`, with lym's bound for the class.
The package imports only the standard library and does no I/O, so the driver, the dobby and tests read the same table.
No other package decides who settles a stop class; the driver's and the dobby's rules for a class lym settles take their bound from the table.

The verdict at a stop is read in this order:

1. `operator` when the row's operator column names the case, the third halt at the row among them.
2. `lym` while lym's settles of the class at the row are within lym's bound.
3. `orch` otherwise.

| Class | lym settles | Orch, `true` | Orch, `unattended` | Operator (severe) |
|---|---|---|---|---|
| Board claim push refused at `Worktree-Create`, a start failing on a held board lock among them | board sync, then re-step; one | settles | settles | third halt at the row |
| Pause covers the run | the dobby resumes it once the pause lifts (D4) | — | — | — |
| Agent launch failed, session timed out, fixer fork failed, strand lost | one fresh start | settles | settles | third halt at the row |
| Stale build | hand-over to a fresh build, as today | when no newer build exists | when no newer build exists | — |
| Escalation `hard-cap`, `circling`, `decision`, `stalemate` | the arbiter, as today | what the arbiter hands on | what the arbiter hands on (D9) | third halt at the row |
| Escalation `budget` | the arbiter (D6) | what the arbiter hands on | what the arbiter hands on (D6) | third halt at the row |
| Bounce budget exhausted | one resume | settles | settles | third halt at the row |
| Gate attempts exhausted | one re-step with a fresh producer session | settles | settles | third halt at the row |
| Webster failure, run-exit audit finding, defect or hub config at a row | — | settles | settles, and files the bug with `lym selfreport create` | third halt at the row |
| Landing stop, deploy failure at `Land` | as today | settles | settles | third halt at the row |
| `driver_not_parked` | the verb waits (D5) | — | — | — |
| Dead driver, agent API error | respawn or revive, within today's caps | after the cap | after the cap | — |
| Provider permission dialog | — | reads and answers | reads and denies when unsure (D9) | — |
| Agent question, entry overtaken, scope change | — | as today's role says | answers, never relays (D9) | `true`: a scope change or a cancel; `unattended`: a run that cannot finish within its record |
| Owner halt, takeover | — | — | — | always |
| Step only the operator can take, no settling action open | — | — | — | always |
| Unnamed stop | the tackler, as today | what the tackler hands on | what the tackler hands on | third halt at the row |

A raddle split's approval, a merge-ready beam and a run at `PR-Review` are waits by design, not stops, and are not in the table (D8).
A stop lym settles within its bound sends the orch no notice; the driver's settle record keeps it.
Every other stop is handed back with its verdict, `orch` or `operator` (D7), so every hand-back to the orch carries `settle: orch`.

Rationale: the operator asked for a rule table that decides per stop class who settles it, with the severe classes the only ones that reach the operator.
A table in a leaf package is the one place that decision lives, so the driver's bounds, the dobby and the orch read one answer.

### D3 — The halt count at a row counts lym's own retries

The halt count at a row is the number of stops of the run at that row, each counted once.
A halt entry in the status history counts as today's class-5 rule counts it: a Stuck at the row with no `routed_to`, plus its `repeats`.
A driver settle record names the halt entry it settled when there is one, and then counts with that entry as one stop;
a settle record counts alone only for a stop that left no halt entry.
A settle counts when its rule retried the row: a re-step after a fault, a fresh start, a resume past a budget, a tackler's resume.
These count nothing:

- a wait: a network wait under `push-retry-not-halt`'s rule, a pause, a `driver_not_parked` wait, a hand-over to a newer build;
- a review escalation the arbiter settles, by resume or accept, since it ends a review round as designed; one the arbiter hands on counts;
- an accept that ends a review segment, since it moves the run on;
- a dead driver or an agent API error, which the dobby reports from its notices and which is no stop at a row.

The count runs over the run's whole history at the row, as today's rule does.
lym's bound for a class (D2) counts lym's settles of that class at the row over the same history,
so the driver's wake restart of its repair count, `wakeRestartGap`, never restores a bound the table set.
One function computes both counts, in `internal/sheddriver`, which owns the settle records.

Bound: a class whose verdict reaches the operator at the third halt at a row does so at the third stop whoever settled the first two,
so a lym retry loop never hides a repeating stop.

Rationale: open point 1 of the study: today's text makes a repeating stop severe at the third halt at a row, and lym's own retries must count toward it.

### D4 — lym settles the mechanical classes within the table's bounds

The driver's rule table, `internal/sheddriver/driverules.go`, gains rows for the classes D2 gives lym, each bounded by the table:

- A board claim push refused at `Worktree-Create`, for a held board lock or a moved ref, gets a board sync and a re-step.
  The refusal is classified as its own class where `internal/boardengine` raises it, so the rule never matches on message text.
- An agent launch that never became ready, a session that timed out, a failed fixer fork and a strand lost from reed get a fresh start of the row's producer.
- A bounce budget exhausted at a row gets a resume, which grants one more budget as the parent's resume does today.
- A Stuck after the row's gate attempts ran out gets a re-step with a fresh producer session.

A run handed back because a pause covers it, a batten step refused for a pause among them, parks as today.
When a pause lifts, the dobby resumes each run whose last stop was that pause's hand-back, as the orch does today on the pause watcher's resume line;
the line then says lym resumed the run.
A run that has moved on since, or that another pause still covers, is left alone.

The tackler keeps every stop no row names.

Bound: each row settles at most its table bound per class at the row (D3); a stop past the bound is handed back with its verdict (D7).
The dobby resumes a paused run only once its pause lifted and only from that pause's hand-back.

Rationale: the study shows the tackler resumed 57 of its 65 sessions; rules settle those classes without an agent and with a bound.

### D5 — `lym loom resume` and `lym loom start` wait for the driver to park

Where `internal/loomcli/resume.go` and `internal/loomcli/start.go` refuse with `shedrun.StartNotParkedKind` because the driver is live and not yet parked,
or its driver lock is held, the verb waits for the park and then acts.
A driver that holds its driver lock and keeps a wait record, such as `push-retry-not-halt`'s network wait, is driving the run, not about to park:
the verb answers at once that the driver drives the run and what it waits on, and changes nothing, since the driver steps again once the wait ends.
The wait wakes at most once a second and backs off while idle, and since an agent blocks on it, its cap comes from the keep-warm resolver.
Past the cap the verb refuses as today, naming the retry as its way forward.
A run lock held on a non-running run is refused at once, as today.
Batten's resume through `internal/battencli/wire.go` keeps today's refusal path and does not wait, so no dobby or batten tick blocks on it.

Bound: the verb waits no longer than the keep-warm cap and changes nothing while it waits.

Rationale: the orch wrapped 16 commands in its own retry loop for this refusal, and the verb can wait for the park itself.

### D6 — The arbiter takes `budget` escalations; open blocking findings never land unattended

`arbiterstop.Eligibility` admits cause `budget` beside `hard-cap`, `circling`, `decision` and `stalemate`.
A `budget` escalation holds an open BLOCKING finding or a red live gate, and the arbiter settles it by these rules:

- With the live gate green and every open BLOCKING finding one the last fixer report marks `deferred`,
  it accepts the round under `false` and `true`;
  at a row past the task's scope, Code-Review and after, it files one board note for the run listing those findings, with `lym board upsert` and kind note.
  Under `unattended` it hands the escalation on.
- Otherwise it buys one more round with `lym loom circling continue`, once per review segment.
- When an open BLOCKING finding or a red gate remains after that round, it hands the escalation on to the run's parent.

Under `true` the orch then accepts or continues, as today.
Under `unattended` the orch never accepts a round with an open BLOCKING finding, deferred or not, or a red gate:
it pauses the run with a note naming the findings and posts a quiet `[operator] held <slug>:` line (D8), and other runs go on.

Bound: the arbiter accepts only a green round whose open BLOCKING findings were all deferred, and never under `unattended`;
it continues at most once per segment; nothing lands a round with an open BLOCKING finding under `unattended`.

Rationale: open point 3 of the study: "accept with a note" for open blocking findings lands known defects unattended, which a landing makes hard to undo;
holding that one run costs only its own time while other work continues.

### D7 — A halt carries its class, verdict and count, and the dobby posts each severe halt to Slack

Every halt the driver hands back records three facts beside the level in the run's stop record, the one the dobby's child read carries:
`class:`, the table's class; `settle:`, the verdict, `orch` or `operator`; and `halts_at_row:`, the count (D3).
The hand-back notice to the orch carries the same three stamps beside `autonomy:` when an orch strand is recorded, as today.

The dobby that drives the run posts each halt whose verdict is `operator`, when the run's level is `true` or `unattended`,
to `<shortname>:SLACK` under its own program name, so the relay routes it to the dobby's orch, the orch of the run's worktree.
The post is one line opening `[operator] severe halt <slug>:`, then the halt, what lym tried and the one action it asks for.
The dobby's state record keeps that the halt was posted, written after the post succeeds;
a dobby restarted between the post and the write posts it once more, which is accepted over losing it.
The post does not depend on a recorded orch strand.
With autonomy `false` nothing is posted to Slack by lym, and the orch relays as today.
The orch relays to Slack only the severe halts it classifies itself, a scope change under `true` and a halt with no settling action open,
and never one whose notice reads `settle: operator`, which lym posted.
The orch's role texts, `role-autonomy.md` and the orch role, say so, and drop "lym classifies no halt".

Bound: lym posts to Slack only a halt the table makes severe, or one D10 escalates, and only under autonomy; every other halt reaches only the orch, as today.

Rationale: a severe halt must reach the operator even when the orch is slow or gone overnight,
and the dobby already posts its wait reminders to the hub's Slack program as a program.

### D8 — A severe-halt post mentions the operator; waits by design stay quiet

The Slack relay's outbound post, in `internal/slackengine/outbound.go`, mentions the configured operator in a post whose first line opens `[operator] severe halt <slug>`,
whoever sent it, lym or an orch, so Slack notifies the operator's phone.
A second such post for the same slug within `slack.remind_after_min` carries no mention, so a relay and a turn reply for one halt notify once.
Every other post, `[operator] split ready`, `[operator] merge ready`, `[operator] held` and every reminder among them, carries no mention.
Slack documents that a user mention, `<@USER_ID>` in the message text, notifies that user (Slack API docs, "Formatting text for app surfaces", Mentioning users).
While the operator is away, the dobby's wait reminders, `internal/dobbyengine/waitreminder.go`, also remind of each run whose severe post its state record holds,
at `slack.remind_after_min`, without a mention, until the run leaves the halt.
A severe halt waits for the operator's answer, which reaches the orch as Slack mail; the dobby goes on starting and driving other runs meanwhile.

Bound: the mention keys on the line prefix alone, so an orch that opens a reply `[operator] severe halt` for a halt outside the classes also notifies;
the role texts permit that prefix only for a severe halt.

Rationale: open point 2 of the study: a raddle approval and a pull request review are waits by design that block no other work,
so they stay quiet, and only a severe halt asks for the operator at once.

### D9 — Under `unattended` the orch never asks the operator

`contracts/orchrole/role-autonomy.md` gains the unattended level's rules, read when a notice carries `autonomy: unattended`:

- The severe classes narrow: a scope change and a cancel are no longer severe.
  A scope change is filed as a board note and the run finishes within its decision record;
  only a run that cannot finish within its record is severe.
- The orch never runs `lym batten cancel`: a run it would cancel, an entry overtaken by landed work among them,
  is paused with a note saying why and posted as a quiet `[operator] held <slug>:` line.
- A design call or circling stop the arbiter hands on is settled with the narrower reading, recorded with `lym loom decision add`.
- An agent's question is answered and never relayed.
- A provider permission dialog is answered only when the orch can read what it allows; when unsure it denies.
- A defect or hub config stop it settles is also filed with `lym selfreport create`.
- A review round is held or accepted as D6 says.

`internal/orchengine/prompt.go`'s autonomy line names the level, so the orch reads which rules apply.

Rationale: the operator decided that under the stricter level the orch never asks the operator and only severe halts go to Slack;
the rules are the "Overnight" column of the study's table.

### D10 — The dobby chases a halt the orch leaves unsettled under autonomy

Under `true` or `unattended`, a run whose stop record reads `settle: orch` and that is still halted at the same stop after `beam.halt_chase_min` (new) minutes,
10 by default, gets one chase line to the dobby's orch naming the halt.
When a parent decision was recorded with `lym loom decision add` after the halt and the run was not resumed, the chase says so and names `lym loom resume` as the way forward.
Under `unattended`, a halt still unsettled one interval after the chase is posted as a severe halt, the class "no settling action open", as D7 posts one,
and D7's state record and D8's reminders cover it.
These settle the halt and end the chase: any transition of the run, a resume, a goto or an accept among them, and a pause covering the run, the orch's hold among them.
The dobby's state record keeps the chase and the post, so a restarted dobby sends neither again.
The key is classified `CategoryNone` in the dobby's `ConfigSpec`, and a value under 1 is refused naming the key, as PATTERN-config-key-names requires.

Bound: one chase per halt, and under `unattended` one severe post per halt after it; a run at a wait by design, or held by a pause, is never chased.

Rationale: the study's recurring operator pushes were a halt the orch had answered without releasing the run and a question nobody acted on;
overnight an orch that stalls must not leave a run halted until morning.

### D11 — The idle notice skips correct waits and names what an agent waits on

The quiet notice in `internal/dobbyengine/runnotice.go`, through `readAgentActivity` in `internal/battencli/wire.go`, reads these as correct waits and raises nothing for them:

- A run at `Raddle-Write` whose proposal passes its checks at the tick and whose raddle agent's session is live, waiting for the operator's approval;
  the dobby's wait reminders also remind of it while the operator is away.
  A proposal that does not pass, or an agent whose session ended, is no wait, so a dead raddle agent still shows.
- An agent holding its turn end with an open question in creel: a message of its own that expects a reply, neither answered nor settled.
  An agent that holds an unread reply is no wait; the notice names the reply it has not read.
  The creel chase still covers a question nobody answers.

An idle notice that still fires names what the agent waits on instead of only "idle":
a live command the agent's pane runs, with how long it has run and when it last wrote output, or the Finalize step a run with no agent is at, with the lock holder or GitHub check it waits on.
The notice "ended a turn without its output files and is held", `internal/shuttleengine/hold.go`, is not sent while the agent has an open question in creel;
every runner that sends it reads that question, not only the ones whose wiring sets a reply probe today.
The verify gate's past-budget notice, `internal/loomshed/verifybudget.go`, names the failing runs, if any, or says none has failed.

Bound: a wait is skipped only while its proof stands at the tick: a passing proposal with a live agent, an open question in creel; a wait with no proof still raises the notice.

Rationale: the study found the quiet notice firing for an operator's approval at `Raddle-Write` and for a held turn end awaiting a creel reply, both correct waits.

### D12 — The dobby warns once when unattended autonomy cannot reach the operator

When a task resolves `beam.autonomy` to `unattended` and the dobby's orch has its slack button off, or the relay has no configured operator to mention,
the dobby posts that orch one line saying severe halts of the task would reach no one, naming `lym slack` as the way forward.
It posts once per episode and again after the cause cleared and returned.

Bound: the warning changes nothing and switches nothing; the relay still drops mail of an orch whose button is off, as today.

Rationale: the relay drops the mail of an orch whose button is off and can mention only a configured operator, so an unattended night without either would hide every severe halt.

### D13 — The glossary names the level and who classifies a severe halt

`internal/heddleengine/glossary.md` gains and changes these lines:

- **unattended autonomy**: autonomy's stricter level, `beam.autonomy: unattended`, in which the orch never asks the operator and only a severe halt reaches them, on Slack. Never: overnight mode, full auto, autopilot.
- **severe halt**: a halt in one of autonomy's closed classes, which the settlement table or the orch classifies and which reaches the operator. Never: critical halt, blocker.
- **settlement table**: the one table in lym that decides, per stop class and autonomy level, whether lym, the orch or the operator settles a stop. Never: rule book, policy.

Rationale: the operator left the level's name open.

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

- `push-retry-not-halt` is at Parent-Align and changes the files D4, D5, D7 and D11 change; this task's runs must start after it lands, so the raddle promotion names it in `depends_on`.
- `worktree-create-transient-net` changes Worktree-Create's stop handling beside D4's board-claim row; the classes differ, a transient `ls-remote` against a refused board claim,
  and the raddle promotion names it in `depends_on` so both rows are built over one stop chain.
- `beam-member-parent` changes which orch a beam member's run reports to; D7's post goes to the orch of the dobby that drives the run, and that task decides whether that orch stays the run's parent.

## Acceptance criteria

- D1: a tier0 test in `internal/dobbyengine` pins that the lookup, the beam config load's check and the local values check accept `false`, `true` and `unattended`,
  that a boolean in a board `run_settings` still reads, and that any other value is refused naming the three values and `lym config beam`.
- D1: a test pins that `hubgeom.Autonomy` returns the level and reads a refused value as `false`, that a halt notice's `autonomy:` stamp carries the level,
  that the board's beam modes show `off`, `on` or `unattended`, and that the dobby posts a refused value once per episode; tier1 or the lowest that fits.
- D1, Review: the template comment on `beam.autonomy` names the three values and says to set `unattended` only once every machine runs a build that knows it, and the orch prompt's autonomy line names the level.
- D2: a tier0 test in `internal/stopclass` (new) pins the table's verdict for every class at each level and at halt counts one, two and three, read in D2's order,
  and that the package imports only the standard library.
- D2: a tier0 test in `internal/sheddriver` pins that each new driver row takes its bound from the table and that a stop past the bound is handed back with `settle: orch`.
- D3: a tier0 test pins the halt count at a row over a history holding a halt entry with repeats, a settle record with no halt entry, a settle record naming its halt entry,
  a network wait, a pause, a build hand-over, an arbiter resume, an arbiter accept and a handed-on escalation, counting each retry once and no wait, arbiter settle or accept.
- D3: a test pins that a class whose verdict is severe at the third halt reaches `settle: operator` when two earlier stops at the row were settled by lym alone,
  and that a wake past `wakeRestartGap` does not restore lym's bound.
- D4: a tier0 test in `internal/sheddriver` pins each new row: a board claim refusal is synced and re-stepped once, then handed back;
  a launch failure, session timeout, fixer fork failure and lost strand get one fresh start, then are handed back;
  a bounce budget exhausted gets one resume, then is handed back; a gate-attempts Stuck gets one fresh re-step, then is handed back.
- D4: a test pins that a stop settled within its bound sends the orch no notice and leaves a settle record.
- D4: a test pins that the dobby resumes a run handed back for a pause once that pause lifts, and leaves alone a run another pause covers or that moved on.
- D5: a test in `internal/loomcli` with an injected clock pins that `lym loom resume` and `lym loom start` wait for a driver that parks and then act,
  answer at once and change nothing for a driver that keeps a wait record, and refuse with `driver_not_parked` naming the retry once the cap passes; tier1 or the lowest that fits.
- D6: a tier0 test in `internal/arbiterstop` pins that cause `budget` is admitted.
- D6: a test pins the arbiter's budget rules: a green round whose open BLOCKING findings are all `deferred` is accepted under `false` and `true`, with one board note at Code-Review and after,
  and handed on under `unattended`; a round with a red gate or an open BLOCKING finding not deferred is continued once per segment and then handed on.
- D6, Review: `role-autonomy.md` says the unattended orch pauses, never accepts, a round holding an open BLOCKING finding or a red gate.
- D7: a test pins that a hand-back records `class:`, `settle:` and `halts_at_row:` in the stop record the dobby reads,
  that the dobby posts one `[operator] severe halt <slug>:` line to `<shortname>:SLACK` under its program name for a severe verdict under `true` or `unattended`, with or without a recorded orch strand,
  that a restarted dobby does not post it again once its state record holds it, and that under `false` nothing is posted to Slack; tier1 or the lowest that fits.
- D7, Review: the orch role texts say the orch never relays a halt whose notice reads `settle: operator`, and no longer say lym classifies no halt.
- D8: a tier0 test in `internal/slackengine` pins that an outbound post whose first line opens `[operator] severe halt <slug>` mentions the configured operator, from an orch and from a program,
  that a second one for the slug within `slack.remind_after_min` does not,
  and that `[operator] split ready`, `[operator] merge ready`, `[operator] held` and a reminder carry no mention.
- D8: a test with an injected clock pins that the wait reminders remind of a run whose severe post the state record holds at `slack.remind_after_min` while the operator is away, and stop once the run leaves the halt.
- D9, Review: `role-autonomy.md` holds the unattended rules D9 lists, the narrowed severe classes among them, and leaves the `true` rules unchanged but for D7's relay rule.
- D10: a test with an injected clock pins one chase after `beam.halt_chase_min` for a run still halted with `settle: orch`,
  the chase naming `lym loom resume` when a parent decision was recorded after the halt, no chase after a transition or once a pause covers the run,
  under `unattended` one severe post one interval after the chase and none for a paused run, and neither sent again by a restarted dobby.
- D10: a tier0 test pins that `beam.halt_chase_min` under 1 is refused naming the key in full.
- D11: a test pins that the quiet notice raises nothing for a run at `Raddle-Write` whose proposal passes with a live agent and for an agent with an open question in creel,
  and still raises it for a `Raddle-Write` run whose proposal does not pass or whose agent ended, and for an agent holding an unread reply, naming that reply; tier1 or the lowest that fits.
- D11: a test pins that an idle notice for an agent running a live command names the command, its run time and its last output time.
- D11: a test pins that the held notice is not sent while the agent has an open question in creel, in a runner whose wiring sets no reply probe today among them,
  and that the verify gate's past-budget notice names the failing runs or says none failed.
- D12: a test pins one warning when a task resolves `unattended` with the dobby's orch's slack button off or no operator configured, none again in the same episode, and one after the cause cleared and returned.
- D13, Review: `internal/heddleengine/glossary.md` holds the unattended autonomy, severe halt and settlement table lines as D13 gives them.

## PATTERN changes

- `PATTERN-stop-settlement` — adds: Deciding who settles a run stop: only `stopclass`'s one table, per stop class and autonomy level; a lym retry counts toward the row's halt count; lym posts only a severe verdict to Slack.

## Notes for the plan writer

- The study's section 4 names where each lym-settled class is handled today; its section 5 names the notices.
- `internal/dobbyengine/waitreminder.go` already posts program lines to the hub's Slack program and keeps its waits in the state record; it is the model for D7's post, D8's reminders and D10's chase.
- The settle records the count reads are written in `internal/sheddriver/tacklerrecords.go`.
- The fixer report's dispositions are defined in `contracts/stencils/burler/burler-step-fix.md`.
- The pause watcher's resume line D4 changes is `queueResumeLine` in `internal/orchengine/pause.go`.
