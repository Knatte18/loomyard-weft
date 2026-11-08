## Goal

An arras family filing turns the run's entry into its project and re-points its dependents

Takes: raddle-perch-mode's `perch` mode and its project outcome whose slug equals the run's entry; the arras recipe of arras-recipe-perch; project-release-members' project release; Board.FileRaddle, checkRunLocks, endsAsSketch and sketchOf, Raddle-File's Call and finish, Board.SetRunStatus and battencli.ClearMotifRun
Delivers: Raddle-File's filing write turning the arras entry into a discussed project holding the whole record, with the part list body and `order`, admitted by the run lock only for that derived shape while the entry's status names Raddle-File; the restore of the entry's record on a failed write; the filing mark under shedrun.RunDir that makes loom's and batten's status writes skip the entry; the store-done and AlreadyFiled repair paths, the halt over a project not filed from the current proposal, and finish skipping EndEntry for a family; ClearMotifRun clearing a done arras run; every dependent's `depends_on` on the arras slug moved onto the filed parts no other part depends on, with a halt naming a refusing entry
Exports: the arras-project admission in boardengine's run lock, the filing mark and its reader in shedrun

## Scope

The part changes these packages:

- internal/boardengine (about 220 changed lines)
- internal/loomshed (about 120 changed lines)
- internal/loomcli (about 60 changed lines)
- internal/battencli (about 80 changed lines)
- internal/shedrun (about 30 changed lines)

## Decisions

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

## Acceptance criteria

- D6: a family arras run leaves the entry as a discussed project under the same slug, holding the whole record, with `order` listing the parts and every part a draft member carrying its `depends_on`.
- D6: a family filing whose board write fails leaves the entry's stored record as it was.
- D6: a write of the converted shape with a changed title or `depends_on`, or made while the entry's run status names a producer other than `Raddle-File`, is refused by the run lock.
- D6: no status write of the arras run after the filing changes the converted project, and a Raddle-File retry after the board write files nothing twice.
- D6: a family filing whose store write fails after the board write, and which then resumes, leaves the converted project with no status and releasable by D8.
- D6: a Raddle-File call over a store-done family filing with no mark writes the mark and leaves the project with no status and not done.
- D6: a Raddle-File call that finds the run's entry already a project not filed from its current proposal files nothing and halts, naming `lym batten cancel` as its way forward.
- D6: `lym beam create` on the converted project clears the done arras run, and the beam run's status writes then reach the entry.
- D8: a family filing leaves every member draft, and a null status written by upsert, merge or batten cancel's claim clear on a converted project releases no member.
- D10: an entry depending on an arras entry depends, after the filing, on the filed task or on the family's last parts, and starts once they have landed.
- D10: a filing whose re-point the board refuses halts the run, naming the refusing entry.
