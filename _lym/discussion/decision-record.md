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
- The arbiter taking review escalations of cause `budget`, and its reading of the autonomy level.
- The dobby's Slack post of each severe halt, the operator mention, and reminders of a severe halt.
- A chase of a halt the orch leaves unsettled under autonomy, of an agent question nobody answers, and of a dead driver past its cap.
- The idle notice's reading of correct waits and its naming of what an agent waits on, and the noisy notices section 5 of the study names.
- The orch role texts, the config comments and the glossary lines the levels change.

Out:

- The network, GitHub API, push lock, failed-push, catch-up and landing network classes:
  `push-retry-not-halt` settles them, by its own rules and bounds, and they are not rows of this task's table.
  A refusal that task stops at once and only the operator can clear, such as refused credentials, is the class "step only the operator can take" (D2).
- Worktree-Create's transient `ls-remote` failure, which `worktree-create-transient-net` owns.
- The parent of a beam member's run, which `beam-member-parent` owns.
- The classes the study's table "Classes that stopped recurring" lists; a recurrence of one reaches the tackler as an unnamed stop does today.
- A hung command or gate inside an agent: section 2 of the study rules it judgment, since telling a hung command from a slow one, and what to kill, needs a reader.
  The idle notice names the live command, its run time and its last output (D11), and the orch judges it.
- A lym runner for a measurement an agent may not run (Rejected alternatives).
- A provider permission dialog: an autonomous session launches with `--dangerously-skip-permissions`,
  and `validatePermissionMode` in `internal/shuttleengine/claudeengine/command.go` refuses the prompt mode on an autonomous run,
  so no autonomous run stops at a dialog, and the table holds no row for one.

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

### D11 — The idle notice skips correct waits and names what an agent waits on

The quiet notice in `internal/dobbyengine/runnotice.go`, through `readAgentActivity` in `internal/battencli/wire.go`, reads these as correct waits and raises nothing for them:

- A run at `Raddle-Write` whose proposal passes its checks at the tick and whose raddle agent's session is live, waiting for the operator's approval;
  the dobby's wait reminders also remind of it while the operator is away.
  A proposal that does not pass, or an agent whose session ended, is no wait, so a dead raddle agent still shows.
- An agent holding its turn end with an open question in creel: a message of its own that expects a reply, neither answered nor settled.
  An agent that holds an unread reply is no wait; the notice names the reply it has not read.
  The creel chase and D10's chase still cover a question nobody answers.

`creelengine.Store.AwaitsReply` answers true both for an open question of the strand's own and while the strand holds an unread reply to one of its messages.
The quiet notice reads it split in two: a probe that answers only an open question of the strand's own,
and a probe of its own for the unread reply, so the notice can name it.
The held notice keeps the combined probe, since an unread reply starts a turn in a live session and the hold sends no notice for it (`internal/shuttleengine/doc.go`).

An idle notice that still fires names what the agent waits on instead of only "idle":
a live command the agent's pane runs, with how long it has run and when it last wrote output, or the Finalize step a run with no agent is at, with the lock holder or GitHub check it waits on.
The notice "ended a turn without its output files and is held", `internal/shuttleengine/hold.go`, is not sent while the agent has an open question in creel or holds an unread reply;
every runner that sends it reads the combined probe, not only the ones whose wiring sets a reply probe today.
The verify gate's past-budget notice, `internal/loomshed/verifybudget.go`, names the failing runs, if any, or says none has failed.

Bound: a wait is skipped only while its proof stands at the tick: a passing proposal with a live agent, an open question in creel; a wait with no proof still raises the notice.

Rationale: the study found the quiet notice firing for an operator's approval at `Raddle-Write` and for a held turn end awaiting a creel reply, both correct waits.

### D12 — The dobby warns once when autonomy cannot reach the operator

When a task resolves `beam.autonomy` to `true` or `unattended` and the dobby's orch has its slack button off, or the relay has no configured operator to mention,
the dobby posts that orch one line saying severe halts of the task, and the stalls D10 posts as severe, would reach no one on Slack, naming `lym slack` as the way forward.
It posts once per episode and again after the cause cleared and returned.
While the button is off the orch relays in its pane a `settle: operator` halt and each stall D10 posts as severe (D7, D10).

Bound: the warning changes nothing and switches nothing; the relay still drops mail of an orch whose button is off, as today.

Rationale: the relay drops the mail of an orch whose button is off and can mention only a configured operator,
so a night under autonomy without either would hide every severe halt lym posts.

### D13 — The glossary names the level and who classifies a severe halt

`internal/heddleengine/glossary.md` gains and changes these lines:

- **unattended autonomy**: autonomy's stricter level, `beam.autonomy: unattended`, in which the orch never asks the operator and only a severe halt reaches them, on Slack. Never: overnight mode, full auto, autopilot.
- **severe halt**: a halt in one of autonomy's closed classes, which the settlement table or the orch classifies and which reaches the operator. Never: critical halt, blocker.
- **settlement table**: the one table in lym that decides, per stop class and autonomy level, whether lym, the orch or the operator settles a stop. Never: rule book, policy.

Rationale: the operator left the level's name open.

### Added after Discussion (parent, 2026-11-27): A stop that leaves no history entry counts once and is recorded (F37)

Decision: Adopt F37's fix: in D3, a stop that leaves no history entry (a producer error such as launch never ready or agent died) counts once too. The driver gives each stepped stop an identity that stays the same across a driver restart; every settle record of it, hand-back and arbiter restep records included, carries that identity; the count at a row is the distinct stops over the halt entries, the handed-on Awaiting entries and the identities of entry-less stops D3 does not exclude. A hand-back or arbiter restep record excludes only a stop already counted or excluded. In D7, the four facts of such a halt live in a stop record the driver writes through internal/state and dobby's child read reads, never in an earlier row's newest entry, and the stop identity is part of the halt key, so two successive failed halts at one row get different keys for D8's dedup, D7's post and D10's chase. Tier0 criteria: an agent launch that fails three times at one row with no history entry (one fresh start, a hand-back, a resume, a third failure) reaches settle: operator; a failed-state hand-back records the four facts, and a second one gets a new key and a second post.

Rationale: Without it the third halt at a row never turns severe for the commonest entry-less stops, which defeats the rule table's bound; the fix stays within D3 and D7.

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

- `push-retry-not-halt` changes the files D4, D5, D7 and D11 change; this task's runs must start after it lands, so the raddle promotion names it in `depends_on`.
- `worktree-create-transient-net` changes Worktree-Create's stop handling beside D4's board-claim row; the classes differ, a transient `ls-remote` against a refused board claim,
  and the raddle promotion names it in `depends_on` so both rows are built over one stop chain.
- `beam-member-parent` changes which orch a beam member's run reports to; D7's post goes to the orch of the dobby that drives the run, and that task decides whether that orch stays the run's parent.

## Acceptance criteria

- D1: a tier0 test in `internal/dobbyengine` pins that the lookup, the beam config load's check and the local values check accept `false`, `true` and `unattended`,
  that a boolean in a board `run_settings` still reads, and that any other value is refused naming the three values and `lym config beam`.
- D1: a test pins that `hubgeom.Autonomy` returns the level and reads a refused value as `false`, that a halt notice's `autonomy:` stamp carries the level,
  that the board's beam modes show `off`, `on` or `unattended`, and that the dobby posts a refused value once per episode; tier1 or the lowest that fits.
- D1, Review: the template comment on `beam.autonomy` names the three values and says to set `unattended` only once every machine runs a build that knows it, and the orch prompt's autonomy line names the level.
- D2: a tier0 test in `internal/stopclass` (new) pins the table's verdict for every class at each level, `false` among them, and at halt counts one, two and three, read in D2's order,
  that no class reaches `operator` at `false`, and that the package imports only the standard library.
- D2: a tier0 test in `internal/sheddriver` pins that each new driver row takes its bound from the table and that a stop past the bound is handed back with `settle: orch`.
- D3: a tier0 test pins the halt count at a row over a history holding a halt entry with repeats, a stop with no history entry and its settle records, a settle record naming its halt entry,
  a network wait, a pause, a build hand-over, an arbiter resume, an arbiter accept, a handed-on escalation's Awaiting entry,
  a halt entry the tackler resumed, with its tackler `spawn` and `restep` settle records and its tackler attempt, which counts once,
  and an interrupted-reinvoke re-step after a driver wake, counting each retry and the handed-on escalation once and no wait, arbiter settle, accept or wake re-step.
- D3: a tier0 test in `internal/sheddriver` pins that a class whose verdict is severe at the third halt reaches `settle: operator` when two earlier stops at the row were settled by lym alone,
  that a wake past `wakeRestartGap` does not restore lym's bound, that a driver restarted at a stopped run keeps the stop's identity,
  and that one that died after a step persisted a second failed stop at the row but before it opened the identity opens a new one;
  and that an agent launch failing three times at one row with no history entry, one fresh start, then a hand-back, a resume and a third failure, reaches `settle: operator`.
- D4: a tier0 test in `internal/sheddriver` pins each new row: a board claim refusal is synced and re-stepped once, then handed back;
  a launch failure, session timeout, fixer fork failure and lost strand get one fresh start, then are handed back;
  a bounce budget exhausted gets one resume, then is handed back; a gate-attempts Stuck gets one fresh re-step, then is handed back;
  and that each row matches its class by kind or field, not by message text, but the bounce budget's row, which matches the `shedengine.ReasonBounceBudgetExhausted` prefix.
- D4: a tier0 test in `internal/sheddriver` pins that a stop settled within its bound sends the orch no notice and leaves a settle record.
- D4: a test pins that the dobby resumes a run handed back for a pause once that pause lifts, matched by the pause entry's scope,
  and leaves alone a run another pause covers, a run that moved on, and a run whose pause the orch set as a hold,
  whose entry stands through a `lym tender resume` with no slug and is lifted by one naming the slug;
  that a hold over the operator's slug pause sets its hold reason on that entry, and a hold over a bolt-hold entry is replaced by a hold entry when the dobby lifts the bolt-hold, neither resuming the run;
  and that the lift resume of a run whose driver has not parked passes the no-wait option, does not block the tick and resumes the run on a later tick; tier1 or the lowest that fits.
- D5: a test in `internal/loomcli` with an injected clock pins that `lym loom resume` and `lym loom start` wait for a driver that parks and then act,
  wait for a run lock held on a non-running run to be released and then act,
  answer at once and change nothing for a driver that keeps a wait record, refuse with `driver_not_parked` naming the retry once the cap passes,
  and refuse at once with the no-wait option batten's child start and resume pass; tier1 or the lowest that fits.
- D6: a tier0 test in `internal/arbiterstop` pins that cause `budget` is admitted and that `arbiterstop.Stop` carries the run's autonomy level.
- D6: a test pins the arbiter's rules: a green `budget` round whose open BLOCKING findings are all `deferred` is accepted under `true`, with one board note at Code-Review and after,
  and handed on under `false` and `unattended`; a missing or older-than-the-round fixer report marks no finding `deferred`;
  a round with a red gate or an open BLOCKING finding not deferred is continued once per segment, read from the segment's circling decision files, and then handed on;
  a segment the Bouncer auto-continued still gets the arbiter's one continue, and a `budget` escalation at the hard cap is handed on, not continued;
  a `budget` escalation over an unreadable ledger with a green gate under `true` is handed on, neither accepted nor continued;
  a `hard-cap` escalation holding an open BLOCKING finding under `unattended` is handed on, not accepted;
  a run halted at Finalize after its fix reviews ran out of rounds with an open BLOCKING finding under `unattended` is held, not accepted;
  and under `unattended` the Bouncer's auto-accept and soft-cap accept pass no round with an open BLOCKING entry; tier1 or the lowest that fits.
- D6, Review: `role-autonomy.md` says the unattended orch pauses as a hold, never accepts, a round holding an open BLOCKING finding or a red gate,
  and a run halted at Finalize after its fix reviews ran out of rounds while one remains.
- D7: a test pins that a hand-back records `class:`, `settle:`, `halts_at_row:` and `halt:` on the halt's history entry the dobby's child read reads,
  that a failed-state hand-back with no history entry records the four on the stop record the child read reads,
  that the next transition's entry or a fold that raises `repeats` supersedes them and gives a new key,
  that a second failed-state hand-back at the row gets a new key and a second severe post when its verdict is `operator`,
  that the dobby posts one `[operator] severe halt <slug> <key>:` line to `<shortname>:SLACK` under its program name for a severe verdict under `true` or `unattended`, with or without a recorded orch strand,
  that a restarted dobby does not post it again once its state record holds it, and that under `false` nothing is posted to Slack; tier1 or the lowest that fits.
- D7, Review: the orch role texts say the orch relays only the severe halts of the rows D2 marks *(orch)*, a cancel under `true` among them, and leaves a halt with no settling action open to lym's post under `unattended`;
  that it never relays a halt whose notice reads `settle: operator` but in its pane while the dobby's orch has its slack button off, where it also relays each stall D10 posts as severe;
  that under `unattended` it holds a run whose halt it relayed as severe, and relays a question that shows a run cannot finish within its record as that run's held severe halt;
  that it still files the bug of a `settle: operator` third halt at a row with `lym selfreport create`; and they no longer say lym classifies no halt.
- D8: a tier0 test in `internal/slackengine` pins that an outbound post whose first line opens `[operator] severe halt <slug>` mentions the configured operator, from an orch and from a program,
  that a second one for the same slug and key within `slack.remind_after_min` does not, that a keyless one dedups only against an earlier keyless one of the slug,
  that a keyless one following a keyed one of the same slug within the window mentions the operator,
  that one for a distinct second halt of the slug within that window does, a fold into the same history entry and a second failed halt at the row with no history entry among them,
  and that `[operator] split ready`, `[operator] merge ready`, `[operator] held` and a reminder carry no mention.
- D8: a tier0 test in `internal/dobbyengine` with an injected clock pins that the wait reminders remind of each severe post the state record holds at `slack.remind_after_min` while the operator is away,
  and stop at the post's own end event: once the run leaves the halt, once a question stall's question is answered or settled, and once a dead-driver stall's driver is revived.
- D9, Review: `role-autonomy.md` holds the unattended rules D9 lists, the narrowed severe classes among them, and leaves the `true` rules unchanged but for D7's relay rule.
- D10: a tier0 test in `internal/dobbyengine` with an injected clock pins one chase after `beam.halt_chase_min` for a run still halted with `settle: orch`,
  for an agent's open question in creel, and for a dead driver past its caps,
  the chase naming `lym loom resume` when a parent decision was recorded after the halt, no chase after a transition, an answered question, a revive or once a pause covers the run,
  under `unattended` one severe post with the stall's key one interval after the chase and none for a paused run, and neither sent again by a restarted dobby;
  that the chase line carries the stall's key, and that with the dobby's orch's slack button off the post's line is also sent to that orch;
  and that a halt the orch relayed as severe under `unattended` and then held gets no second severe post.
- D10: a tier0 test pins that `beam.halt_chase_min` under 1 is refused naming the key in full.
- D11: a test pins that the quiet notice raises nothing for a run at `Raddle-Write` whose proposal passes with a live agent and for an agent with an open question in creel,
  and still raises it for a `Raddle-Write` run whose proposal does not pass or whose agent ended, and for an agent holding an unread reply, naming that reply; tier1 or the lowest that fits.
- D11: a test pins that an idle notice for an agent running a live command names the command, its run time and its last output time; tier1 or the lowest that fits.
- D11: a test pins that the held notice is not sent while the agent has an open question in creel, in a runner whose wiring sets no reply probe today among them,
  nor while it holds an unread reply, and that the verify gate's past-budget notice names the failing runs or says none failed; tier1 or the lowest that fits.
- D12: a tier0 test in `internal/dobbyengine` pins one warning when a task resolves `true` or `unattended` with the dobby's orch's slack button off or no operator configured,
  none again in the same episode, and one after the cause cleared and returned.
- D13, Review: `internal/heddleengine/glossary.md` holds the unattended autonomy, severe halt and settlement table lines as D13 gives them.

## PATTERN changes

- `PATTERN-stop-settlement` — adds: Deciding who settles a run stop: only `stopclass`'s one table, per stop class and autonomy level; a lym retry counts toward the row's halt count; lym posts only a severe verdict to Slack.

## Notes for the plan writer

- The study's section 4 names where each lym-settled class is handled today; its section 5 names the notices.
- `internal/dobbyengine/waitreminder.go` already posts program lines to the hub's Slack program and keeps its waits in the state record; it is the model for D7's post, D8's reminders and D10's chase.
- The settle records the count reads are written by `AppendSettleRecord` in `internal/sheddriver/settle.go`, the tackler's among them.
- The fixer report's dispositions are defined in `contracts/stencils/burler/burler-step-fix.md`.
