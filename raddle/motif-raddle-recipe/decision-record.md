# Arras: one recipe that runs Motif and raddle in a row, with a perch on the split

## Goal

One loom recipe, `arras`, takes a tapestry-sized board entry from Discussion to drafted tasks with no operator step in between.
It runs the Motif block (Discussion and its perch), then the raddle split, then a review perch on the split, then files the parts as drafts.
The arbiter settles the perch's stops, so only what it cannot settle reaches the orch.
Operator decisions made after the Discussion enter the decision record through `lym loom decision add --by operator`, so the split and its perch see them.
The operator's one remaining step is the release of the filed tasks.

## Scope

In:

- The `arras` recipe file, its registration as a recipe a board task may name, and its loom keys and profile blocks.
- The split perch: a Bouncer ring over the raddle proposal, with a new rubric stencil.
- A raddle mode for a run whose recipe holds the split perch, which files with no operator approval.
- The arras filing outcomes: one drafted task, or a project under the arras entry's slug holding the drafted family.
- The run's own entry turned into that project in the filing write, and the paths after the filing that touch it.
- The correction of `freshRaddleEntry`'s goto test, which serves the `raddle` recipe too (D4).
- Entries that depend on the arras entry, re-pointed at the filed work.
- The release of a project releasing its draft member tasks.
- The shipped text: `plugins/ly/skills/board-policy/SKILL.md`, `plugins/ly/skills/board/SKILL.md`, `contracts/orchrole/orch-role.md`, the raddle stencil's mode text, and `docs/reference/weaving-glossary.md`.

Out:

- The `motif` and `raddle` recipes keep their behaviour: `motif` still makes a Sketch, and `raddle` still asks the operator unless `raddle_auto_approve` is set and keeps all four shapes.
  The one change a `raddle` run sees is D4's correction of the goto into Raddle-Write.
  D5 limits only the top-level shape an arras run files; each part's recipe and profile stay raddle's choice.
- No closed project whose darns skip their own perch; each filed task is an ordinary darn or tapestry run with its own perch and its own landing.
- No change to the hub's `default_recipe`: D9's default is guidance in the shipped text.

## Decisions

### D1 — The `arras` recipe is the Motif block followed by raddle's write and file rows and the split perch

Decision: `contracts/recipes/arras-recipe.yaml` (new) declares `contract: loom` in its record-only form.
Its rows, in on_done order:

1. A block row `Motif` with `block: motif`, `block_gate: undiscussed` and `on_done: Raddle-Write`.
2. `Raddle-Write`, engine `RaddleWrite`, with its `raddle` gate.
3. The split perch of D2.
4. `Raddle-File`, engine `RaddleFile`, its `on_stuck` returning to `Raddle-Write` as in `contracts/recipes/raddle-recipe.yaml`.

It holds no `Record-Write` row: the Motif block's `DiscussionWrite` is the one record row the loom contract allows, and Raddle-Write reads the same decision record path either way.
The `undiscussed` gate seeds the stored record and skips Discussion when the entry is already discussed, so an arras run restarted after its Discussion approval resumes at Raddle-Write.
`arras` is registered wherever `motif` and `raddle` are, and the test case that pins `arras` as an unknown former recipe name is reversed.

Rationale: the Motif block is already how tapestry runs Discussion, so arras reuses both sub-flows without copying rows, and the loom contract needs no change.

### D2 — The split perch is a Bouncer ring over the proposal, with a cold fixer

Decision: three rows in a review segment of their own, `Raddle-Bouncer`, `Raddle-Burler-Review` and `Raddle-Burler-Fix`, built on the generic `Bouncer`, `BurlerReview` and `BurlerFix` engines.

- The artifact is `_lym/raddle/proposal.yaml`; the decision record is read-only context named in the rubric.
- The rubric is the new stencil `contracts/stencils/loom/loom-rubric-raddle-review.md` (new), embedded and registered like the other loom rubrics.
- The fixer is a cold `BurlerFix` session with `fix-scope: overlay` and the `raddle` gate, so a fixed proposal still passes every proposal check before the next round.
- The Bouncer sets no `carry_over`, so no round writes into the decision record, and no `approve_seam`, so the Discussion approval stays the recipe's one approval row.
- The Bouncer sets no `commit_seam`: Raddle-File's commit of the raddle directory commits the approved proposal.
- `escalate_decisions` is on, so decision-class findings go to the run's parent through the Bouncer, where the arbiter settles them first under its existing rules; the fixer row sets `decisions_escalated`.
- The segment takes its own loom keys, in the shape the Discussion segment's keys take: a bounce budget, reviewer and fixer fans, an auto-accept setting and reviewer, fixer and judge model keys.
  The fixer's model key defaults to the raddle key's model, so the split is fixed by a model of the raddle agent's strength.
  The split perch's auto-accept key, `<perch>_auto_accept` under its perch name, defaults to false and is a guard-category key, as `reviewbudget` registers every perch's auto-accept key.
- A fresh entry into Raddle-Write, a return from Raddle-File or a goto into the row, re-arms the perch when the perch's last verdict settled,
  as after a Raddle-File return or a goto from Raddle-File.
  The split perch's Bouncer has no Subject seam, so it keeps no settle or generation record,
  and its next entry over that settled verdict clears its run directory.
  The new proposal is then reviewed in a fresh cycle with the segment's full budget.
  Raddle-File's halt on its third failure bounds the cycles a run can spend.
- A goto into Raddle-Write from a perch that has not settled, such as one awaiting its parent on an escalation, opens no fresh cycle,
  and that is accepted.
  The Bouncer finds its recorded unsettled verdict, so it clears nothing and settles that round again, raising its escalation again.
  Shed's goto keeps the spent bounce count, and with no Subject seam no seed starts a new generation.
  Below the hard cap, `lym loom circling continue` runs the next round over the new proposal.
  At or past the hard cap the round can only be accepted,
  so `lym loom circling accept` files the new proposal with no round having reviewed it, as an accept below the cap does too.

Rationale: the Bouncer ring is the one perch mechanism the arbiter already serves, and a cold fixer reads only the proposal, the record and the findings.

### D3 — The split rubric judges the split against the whole record

Decision: the rubric directs the reviewer to read the decision record, every `### Added after Discussion` entry included, and to judge the split's judgement, not its format:

- Every decision, Added after Discussion entry included, lands in a part whose scope and contract can build it.
- The seams: each part's contract (takes, delivers, exports) matches what the parts on its `depends_on` path deliver.
- The order follows the record's stated order, or the departure is listed in `order_departures` with its cost.
- Each part's recipe and profile fit its size and risk, and the cost lines rest on the cells they cite.
- One task or a family fits the record's scope.

The mechanical checks of `contracts/specs/raddle-proposal-spec.md`, D5's outcome checks included, are named as already enforced by the gate and are not re-derived.

Rationale: an Added after Discussion entry carries no decision id, so the mechanical checks cannot tell whether the split builds it.

### D4 — An arras run files on the perch's approval, with no operator approval

Decision: Raddle-Write records a third raddle mode, `perch`, in the raddle state on its first call when the recipe's rows hold the split perch, and then does not read `raddle_auto_approve`.
The mode is the one signal of an arras run: the `raddle` gate, `lym loom raddle-propose`, `lym loom raddle-approve` and Raddle-File read it from the state.
`lym loom raddle-approve` refuses in `perch` mode, naming the perch as the approval.
The perch's approved settle routes to Raddle-File, which files with no approval-hash match, as in auto mode.
In `perch` mode Raddle-Write ends on the finished proposal and the raddle gate's pass stores the cold proposal, as in auto mode.
Every fresh entry into Raddle-Write, a return from Raddle-File or a goto into the row, moves the last proposal aside, as in auto mode.
`freshRaddleEntry` in `internal/loomcli/raddlewiring.go` is corrected to match a goto entry as `shedengine.Goto` and `shedengine.Relaunch` write it:
the target row as its producer, outcome `goto` and no `RoutedTo`.
Its goto branch tests that the last entry's outcome is `goto` and its producer is `Raddle-Write`;
the Raddle-File Stuck branch keeps its `RoutedTo` test.
The goto rows of its table test in `internal/loomcli/raddlewiring_test.go` are rebuilt the same way,
since they carry producer `Raddle-File` and a `RoutedTo`, an entry no verb writes.
Without the correction a goto into the row is never fresh, so the seat's new session starts over the old proposal,
in a `raddle` run as in an arras run.

Bound: this removes the operator's approval only in a run whose recipe rows hold the split perch.
Raddle-File still runs every proposal check and the board re-check before and inside its write, and every filed task stays draft until the operator releases it.
Two routes file a proposal whose last round still had open findings.
With the split perch's auto-accept key on, a spent budget accepts such a round.
That route is bounded by the key's default of false and its guard category, which marks it as a key that hollows an approval.
With the key off, the Bouncer's soft-cap rule (`softCapRule` in `internal/shedadapters/bouncer.go`) settles a spent-budget round:
it accepts automatically a round in which no open BLOCKING or MEDIUM ledger entry carries a fix tag other than `mechanical`, a missing tag included,
as `designWrongKeys` in `internal/shedadapters/bouncerfiles.go` counts them,
and a round with such an entry continues automatically or escalates at the hard cap or on circling.
The accepted round may still hold open LOW and NIT entries of any tag and open BLOCKING or MEDIUM entries the judge tagged `mechanical`,
and the proposal is filed with them;
the judge's tag is the only thing between a BLOCKING `mechanical` entry and the filing.
That route is bounded by the judge's fix tags, the soft-cap window and the automatic flag on the round's decision file.
Since D2 sets no `carry_over`, the open findings of an automatically accepted round are dropped from the record;
they stay only in the round's ledger in the perch's run directory.
Raddle-File's proposal checks and the filed tasks' draft status hold on both routes.
D2's goto from a perch that has not settled is a third route, which can file a proposal no round reviewed;
the same checks and draft status hold on it.

Rationale: the perch replaces the manual approval the operator kept only while raddle was being tuned; release stays the operator's.

### D5 — An arras run ends in one drafted task or one project of drafted tasks

Decision: in `perch` mode the proposal checks allow two outcomes:

- One task: shape `tapestry`, or shape `tasks` with one part.
  The task is filed draft with its record, and the arras entry ends done through `boardengine.Board.EndRaddleEntry`, so the landing's board prune removes it.
- A family: shape `project`, whose `project.slug` equals the arras entry's slug.
  Each part is filed draft as a member of that project, with its `depends_on` and its part of the record.

Shape `beam`, and shape `tasks` with more than one part, fail the checks in `perch` mode with a finding naming the allowed shapes.
The `slug-sketch` and `slug-taken` checks exempt a `perch`-mode project slug, which must equal the run's entry.
The raddle stencil's mode text for `perch` mode tells the agent these outcomes.

Rationale: the operator set these two ends; a beam would start the family's runs, which would take the release from the operator.

### D6 — The family's project is the arras entry itself, turned into a project in the filing write

Decision: for the family outcome, Raddle-File's one board write turns the run's own entry into the project instead of ending it done:

- The entry becomes kind `project`, loses its run status and the fields only a run or the dependency graph reads, keeps its slug, title and labels, and stays discussed.
- Its `order` lists the parts in their filed order, which follows `depends_on`.
- Its stored record is the whole Sketch record, as raddle already stores on a filed project.
- Its body opens with the raddle project body prefix and the part list, followed by the entry's former body.
- When the write fails, the record file the entry held before the write is restored, never removed with the filed parts' record files.

The run lock admits the change only when the entry before the write holds a run status whose producer is `Raddle-File`,
and the entry after the write equals the arras project derived from the entry before it and the filed parts.
The lock's input is the `after` list that `checkRunLocks` in `internal/boardengine/runlock.go` receives.
The filed parts are its entries absent from the loaded list whose `project` is the slug, each a discussed draft task,
taken in the sequence they appear in `after`; the lock never reads them from the entry's own `order`.
The filing write adds the parts in their filed order.
Before the filing write, Raddle-File writes its own run status on the entry when the status names another producer,
so a status write that loom's hook failed at the move into the row cannot make the run lock refuse the filing.
This mirrors `endsAsSketch`, which admits a Sketch only when the entry equals `sketchOf` of the entry before:

- kind `project` and no status;
- cleared, the fields a run, a landing or the dependency graph reads: the set `boardengine`'s `sketchOf` clears (`depends_on`, `isolated`, `recipe`, `profile`, `run_settings`, `pull_request`, `notify_on_land`);
- unchanged, the title, labels, `project`, `beam` and discussed flag;
- the body exactly the raddle project body prefix for the slug, the part list and the former body, the part list naming the filed parts in their sequence in `after`;
- `order` exactly the filed parts in their sequence in `after`, with each part after the in-family parts it depends on.

The lock recomputes no canonical dependency order:
`dependencyOrder` in `internal/loomshed/raddlefile.go` keeps the proposal's order for ties, and the lock never sees the proposal.

Bound: the admission lets through only that change,
and only while the entry's run status has `Raddle-File` as its producer half, whatever its state half,
so a run halted at Raddle-File keeps the admission open until its entry changes.
A write that makes any other scope change alongside it, such as a new title, labels, `depends_on` or body text, is refused by the run lock.
The lock reads the entries before and after the write and no caller, so any board writer that makes this write passes and ends the run's hold,
and the lock cannot tell the run's approved parts from any other discussed draft tasks the write adds.
That is accepted, since the admitted write leaves only draft discussed members, the fixed project body, no released task and no run,
so the operator's release stays the gate.

Raddle-File's `Call` finds a filing as its own only when the store holds its filing,
or when its stage holds the proposal's hash and the board holds every slug of the run's own proposal as filed from the Sketch (`AlreadyFiled`).
Both tests key on the current proposal's hash.
Otherwise `Call` runs the proposal checks, and the run's entry is already a project when they run
after a filing another writer made with other slugs, or after the run's own filing when a goto has since re-split the proposal.
So Raddle-File reads the run's entry before its checks.
When the entry is already of kind `project` and neither test holds, it files nothing and halts for the parent, never sending the proposal back,
naming the entry's `order` as a filing not made from the current proposal.
Its way forward is `lym batten cancel <slug>`, which ends the run and leaves that filing for the operator to keep, edit or remove.

After the filing, the arras run's own status writes, loom's and batten's, leave the board entry alone.
The skip is keyed on a filing mark among the run's records under `shedrun.RunDir`, beside its status and child-recipe records, which Raddle-File writes when its board write lands with a project.
The mark goes with the run's records when the arras extension of `battencli.ClearMotifRun` clears them, so the skip ends there.
A crash or a failed store write between the board write and the mark lets loom's status hook write a run status onto the converted project on the resume.
So whenever a Raddle-File call finds the converted entry filed as its own, by either test above,
it writes the mark and then clears any run status on the entry at the start of `Call`, whether or not the store is done,
and then writes the store when it is not done.
When the store is done, `Call` skips the filing steps and goes straight to `finish`, so neither write sits in the store-writing path.
In the family outcome `finish` skips `EndEntry` on every path, retries included,
since `EndRaddleEntry` in `internal/boardengine/raddlefile.go` would mark the converted project done.
Each step is idempotent, so a crash between them is repaired by the next retry.
`lym beam create` on the project clears the done arras run under its slug, as it clears a done motif run, by extending the recipe test in `battencli.ClearMotifRun`.

Rationale: the project carries the arras entry's slug so the board shows the family as that run's result, and a slug cannot hold two entries.

### D7 — Decisions made after the Discussion enter the record through `lym loom decision add`

Decision: during an arras run past the Discussion approval and before Raddle-File, an operator decision is recorded with `lym loom decision add --by operator`.
It appends to the run's decision record and stores it on the board, as for a run of any recipe past that approval.
Raddle-Write reads the record at its start, the perch reads it every round, and every filed part's record carries every Added after Discussion entry whole through `discussionparser.SplicePart`.
The orch text says an operator decision about a running arras entry goes in this way and never into the entry body, which the run lock holds.

Bound: from the Raddle-File row on, `lym loom decision add` on a run whose raddle state records `perch` mode refuses.
D4 records that mode from the recipe's rows, so the refusal is decided by the rows and a `raddle` run keeps today's behaviour.
Before the filing its way forward is `lym loom goto --to Raddle-Write`, which re-splits with the decision recorded first.
After the filing the refusal lists the filed slugs, read from the run's filing.
Its way forward is `lym loom decision add` on the part the decision concerns, once the operator has released that part and its run has passed its record row.

Rationale: the `slack-v2` miss came from decisions left in the entry body; the record is the one place the split and its perch read.

### D8 — Releasing a project releases its draft member tasks

Decision: `lym board set-status` with a null status on a project whose own status is empty releases, in the same write, every task whose `project` is that project and whose status is `draft`.
Dobby then starts them in their `depends_on` order, as it starts any released task.
The project keeps the whole decision record, which raddle already stores on a filed project.

Bound:

- The trigger is that verb alone.
  The release lives in the verb's own path or a board method only the verb calls, never in `Board.SetStatus` or `Store.SetStatus`,
  so their other null-status callers, batten cancel's claim clear among them, release no member.
  An upsert or a merge that writes a null status, and D6's filing write, release no member either.
- Only direct member tasks in `draft` are released; a member already released, running or done, a nested project and its members, and entries outside the project are untouched.
- A project whose status is a run status or `done` is refused, naming the release of each member by its slug as the way forward.
- The write validates each released member as its own release would and refuses as a whole when one fails.

Rationale: the operator's one step after an arras run is the release; one command releasing the family keeps it one step.

### D9 — The shipped rule names `arras`

Decision: the shipped instructions that pick a recipe for a board entry name `arras` as the recipe for a tapestry-sized note.
`motif` alone stays for a design decided now but split later, on code still changing.
`plugins/ly/skills/board-policy/SKILL.md` and `contracts/orchrole/orch-role.md` carry the rule and the release of a project as one release.
The rule this rewrites is the Motif-first rule that commit c8e43ac4 landed on main in `plugins/ly/skills/board-policy/SKILL.md`,
so the run takes main's text of that file before editing it.
`plugins/ly/skills/board/SKILL.md` describes the arras run.
`docs/reference/weaving-glossary.md` lists `arras` as the recipe and keeps `cartoon` reserved.

Rationale: the operator named `arras` the default for a tapestry-sized note, and the rule belongs to lym for every hub.

### D10 — Entries that depend on the arras entry wait on the filed work

Decision: the filing write moves every other entry's `depends_on` on the arras slug onto the filed parts that no other filed part depends on.
In the one-task outcome that is the one filed task; in the family outcome it is the family's last parts.
A board refusal of the re-point, such as a run lock on a dependent, halts the run for its parent, naming the refusing entry and the refusal as its way forward.
It never returns the run to Raddle-Write, since no new split clears a cause on another entry.

Bound: only `depends_on` entries naming the arras slug change, and only in the filing write.

Rationale: in the one-task outcome the arras entry ends done, so a dependent left on it would start before the filed task has landed.
In the family outcome a dependent left on the converted project would never start:
each landing prunes the done member, and `boardengine.Dependencies` never counts an open project with no member left as met.

## Constraints

- `PATTERN-shed-recipe-registry`: arras declares `contract: loom`, and a verb that serves some loom recipes refuses on others by the recipe's rows, never its name (D4's mode, D7's refusal).
- `PATTERN-stencil-ownership`: the new rubric and the raddle stencil's mode text are read at call time through `stencilstore`.
- `PATTERN-sole-parsers`: D5's outcome checks live in `internal/raddleparser`, the proposal's sole parser, which takes the mode as an option.
- `PATTERN-refusal-way-forward`: the refusals of D4, D5, D7 and D8 name their way forward, and a test reaches each.
- `PATTERN-run-records`: post-Discussion decisions are written only by `lym loom decision add`.
- `PATTERN-dobby-runs`: released members start only through their parent's dobby, which D8 relies on and does not change.

## Auto-mode assumptions

- The perch's fixer is cold, not Raddle-Write re-entered (D2).
- A project release releases its draft members at once, and the project keeps the whole record (D8).
- Shape `beam` is refused in an arras run (D5).
- The family's project is the arras entry turned into a project (D6).

## Open risks

- A decision added after the perch's last approving round and before Raddle-File reaches every part's record but is not reviewed against the split's seams.

<!-- lym:carry-over Discussion-Review -->
- Discussion-Review, round 3: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/discussion/round-3-review.md; fixer report: _lym/reviews/discussion/round-3-fixer-report.md.
  - d6-admission-unbounded (class design, severity MEDIUM, open in rounds 1, 2, 3)
  - goto-raddle-write-no-rearm (class design, severity BLOCKING, open in rounds 2, 3)
  - rearm-keeps-generation-record (class design, severity MEDIUM, open in rounds 2, 3)
  - split-perch-auto-accept-unbounded (class design, severity MEDIUM, open in rounds 1, 2, 3)
<!-- /lym:carry-over Discussion-Review -->

## Acceptance criteria

- D1: an `arras` task runs Discussion, its perch, Raddle-Write, the split perch and Raddle-File in that order, and the loader accepts the recipe under the loom contract.
- D1: an arras run restarted after its Discussion approval skips Discussion and resumes at Raddle-Write with the stored record.
- D1: `arras` is accepted as a board task's recipe and as a task worktree's run recipe.
- D2: the split perch reviews `_lym/raddle/proposal.yaml`, a fixed proposal that fails the `raddle` gate is not handed to the next round, and decision-class findings reach the run's parent through the Bouncer.
- D2: no split perch round changes the decision record, and `lym loom decision add` still finds the Discussion approval row in an arras run.
- D2: after a Raddle-File return or a goto to Raddle-Write from Raddle-File, the new proposal is reviewed by the perch in a fresh cycle with the segment's full budget before Raddle-File runs.
- D2: after a goto to Raddle-Write from a perch awaiting its parent on an escalation below the hard cap, the perch raises that escalation again, and `lym loom circling continue` then runs the next round over the new proposal.
- D2: with the split perch's auto-accept key off, the Bouncer never settles as an accept a spent-budget round holding an open BLOCKING or MEDIUM ledger entry not tagged `mechanical`: it continues automatically or escalates to the run's parent.
- Review: D3 — the rubric names the decision record and its Added after Discussion entries as the authority and names each judgement D3 lists.
- D4: an arras run records `perch` mode and reaches Raddle-File on the perch's approval with `raddle_auto_approve` off, and `lym loom raddle-approve` on it refuses naming the perch.
- D4: a `raddle` run keeps its interactive approval when `raddle_auto_approve` is off.
- D4: an arras run's Raddle-File finds a stored cold proposal after the perch's approval, also after a return.
- D4: with the split perch's auto-accept key off, a spent-budget round whose open BLOCKING and MEDIUM entries are all tagged `mechanical`, beside an open LOW entry of another tag, is accepted automatically, its decision file marked automatic, and Raddle-File files its proposal.
- D4: after `lym loom goto --to Raddle-Write` on an arras run, the last proposal is moved aside and the seat starts a new session.
- D5: in `perch` mode a proposal of shape `beam`, of shape `tasks` with two parts, or of shape `project` whose project slug differs from the entry's slug fails its checks, and the fixer's gate run applies the same checks.
- D5: a one-task arras run files one draft task with its record and leaves the arras entry done.
- D6: a family arras run leaves the entry as a discussed project under the same slug, holding the whole record, with `order` listing the parts and every part a draft member carrying its `depends_on`.
- D6: a family filing whose board write fails leaves the entry's stored record as it was.
- D6: a write of the converted shape with a changed title or `depends_on`, or made while the entry's run status names a producer other than `Raddle-File`, is refused by the run lock.
- D6: no status write of the arras run after the filing changes the converted project, and a Raddle-File retry after the board write files nothing twice.
- D6: a family filing whose store write fails after the board write, and which then resumes, leaves the converted project with no status and releasable by D8.
- D6: a Raddle-File call over a store-done family filing with no mark writes the mark and leaves the project with no status and not done.
- D6: a Raddle-File call that finds the run's entry already a project not filed from its current proposal files nothing and halts, naming `lym batten cancel` as its way forward.
- D6: `lym beam create` on the converted project clears the done arras run, and the beam run's status writes then reach the entry.
- D7: `lym loom decision add --by operator` during an arras run's Raddle-Write row appends the entry, stores the record on the board, and every filed part's record carries it.
- D7: `lym loom decision add` on an arras run at Raddle-File or later refuses with its way forward: before the filing the goto to Raddle-Write, after it the filed slugs and the record step on a released part.
- D7: `lym loom decision add` on a `raddle` run at Raddle-File behaves as it does today.
- D8: `lym board set-status` with a null status on a project releases its draft member tasks in one write and leaves released, running and done members and nested projects untouched.
- D8: a family filing leaves every member draft, and a null status written by upsert, merge or batten cancel's claim clear on a converted project releases no member.
- D8: a project release whose member fails validation releases none, and a project with a run status or `done` is refused.
- D10: an entry depending on an arras entry depends, after the filing, on the filed task or on the family's last parts, and starts once they have landed.
- D10: a filing whose re-point the board refuses halts the run, naming the refusing entry.
- Review: D9 — the board-policy skill, the board skill, the orch role and the weaving glossary name `arras` as the recipe for a tapestry-sized note and keep `motif` alone for a split made later.

## Notes for the plan writer

- Raddle-File's return to Raddle-Write relies on a shared segment in `contracts/recipes/raddle-recipe.yaml`; place the split perch's segment so that return stays legal.
- `finishesAsSketch` in `internal/loomcli/wiring.go` already keeps an arras run from making a Sketch, since it checks for a `RaddleFile` row.
