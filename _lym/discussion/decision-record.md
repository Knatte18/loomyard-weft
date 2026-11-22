# Move a running task by a move note and a fresh run

## Goal

Give the orch one written procedure for moving a running task to another parent:
the orch holds the run, its live writer writes what it learned to a move note,
the orch cancels the run with `lym batten cancel <slug> --remove`,
then writes the note into what the entry's fresh run reads, sets the entry's new parent and has it released.
The fresh run reads the old run's commits and never re-uses them.

## Scope

In:

- A new section of the orch's beam role file, `contracts/orchrole/role-beams.md`, holding the procedure of D1 to D7.
- A pointer to it in the run loop of `contracts/orchrole/orch-role.md`, beside the cancel lines, and in that file's line naming what `role-beams.md` covers.
- One glossary line in `internal/heddleengine/glossary.md` (D8).

Out:

- Any new `lym batten` verb, flag or recipe row (D1).
- Any Go change to cancel, the board, fabric or the archive tags.
- Moving a beam project, a grouping project or a front task.

## Decisions

### D1 — Orch guidance alone, built on existing verbs

The move is a procedure in the orch's role text, run by the orch with verbs that exist today:
`lym batten pause`, `lym creel post` to the writer, `lym batten cancel <slug> --remove` from the prime,
and `lym board upsert` and `lym board set-status` on the entry.
No batten verb, flag or recipe row is added, and it is never a recipe step.

Rationale: moving a running task is rare, and every step but the note is an existing verb.
The note is written by an agent session, which no verb can drive synchronously,
so a verb would only wrap a cancel and an upsert the orch already runs.
Bound: the procedure adds no path past a guard;
cancel keeps all its refusals, and the board accepts the edit only because cancel set the entry back to `draft`, which clears the run lock.

### D2 — Hold the run, then the writer writes the move note

The orch first holds the run with `lym batten pause <slug>`, so no row, batch commit or Parent-Align merge-in moves the task branch while the note is written and read.
It then asks the run's live writer, by `lym creel post --expect-reply <strand>`, to write a move note at `_lym/move-note.md` in the task worktree and to reply when done.
The live writer is the session of the row the run is in: the Discussion writer, the plan writer, or the Webster master session during Webster.
The note holds four sections:

- what is done, naming the cards or decisions it covers;
- the decisions taken, each with its reason;
- what is left;
- where the commits are: the task branch, its tip SHA and its merge base with the old parent branch.

When the writer replies, the orch reads the note and copies it to its own `.scratch/` before the cancel.
The note is not committed, so the records archive tag does not reliably hold it, departing from the entry's "the archive tag keeps the hand-back";
the board copy of D4 is the durable one, and the orch's scratch copy covers the span between the cancel and the board write.

Rationale: the entry is held by the run and takes no body edit, so the worktree is the only place the writer can put what it knows.
The pause keeps the note's tip SHA and merge base equal to what the cancel archives.
Committing the note would need a new committing writer for no gain over the board copy.

### D3 — No live writer

When none of D2's writers is live, because the run is halted, sits in a review, gate or landing row, or its writer's strand is gone,
the orch writes the move note itself.
It reads the run's decision record, plan and status, and the task branch's log against the old parent, and fills the same four sections.

Rationale: the procedure must not depend on a writer the orch cannot bring back without resuming the run it means to end.

### D4 — Order: hold, note, cancel, carrier, parent, release

1. The orch pauses the run and the writer writes the move note (D2), or the orch does (D3).
2. The orch runs `lym batten cancel <slug> --remove` from the prime, on the entry's owner machine.
   It keeps the code archive tag from the command's `archive_tags`; when the list is empty, the carrier names no tag.
   A cancel that reports a partial finish is run again until it completes, before step 3.
3. The orch upserts the same entry (D5), writing a move note section into what the fresh run reads:
   - an entry with no stored decision record gets a `## Move note` section in its body;
   - an entry with a stored decision record (`discussed`) runs from that record, never its body,
     so the orch adds the section under the record's `## Notes for the plan writer` and stores it with `lym board upsert --record-file`.
   The section holds the note, the code archive tag, the records archive tag's pattern `archive/<slug>/` in the records repository, and D6's read-only rule.
   In the same upsert, `project` is set to the new beam's slug, or emptied for the root.
4. The entry is released with `lym board set-status` and a null status by the orch whose worktree is the new parent.
   When the new parent is another orch's worktree, the moving orch leaves the entry `draft` and tells the operator which orch releases it.

Rationale: `--remove` is the only cancel that makes archive tags and lets the slug run again (Constraints).
Releasing stays with the orch that owns the new parent, as the orch role already says for every entry.

### D5 — The same entry carries on, not a new task

The moved work stays on the same entry and slug; the orch files no new task for it.

Rationale: after `--remove` the entry is a `draft` with no pair, and nothing but its `project` field records a parent,
so changing `project` is the whole move on the board side.
The entry keeps its handle, history, labels and run settings.

### D6 — The fresh run reads the old commits, never re-uses them

The fresh run forks from its new parent's head like any run.
It reads the old work through the code archive tag: the diff from the note's merge base to the tag.
No agent cherry-picks, merges or checks out the archive tag into the new task branch;
work the fresh run keeps is written again through its own plan and gates.
A move before any code commit leaves an empty diff, and the note alone carries the old run's work.

Rationale: a cherry-pick or merge of the old tip brings back the study's ancestry question (`docs/research/loom/reparent-running-task-2026-11-12.md`, section 3),
since the old tip can carry unlanded commits of the old beam into a parent that never took them.
Reading the diff keeps what was learned and puts every kept line through the new run's review.

### D7 — When a move is refused

The procedure moves a task only through a cancel that completes.
When cancel refuses, the move stops there: the orch lifts its pause with `lym batten run <slug>`, and takes the way forward cancel's message names;
for work that already reached its parent, that is letting the landing finish, after which there is nothing to move.
A beam project, a grouping project and a front task are never moved this way.

Rationale: cancel's refusals already guard the states where a move would lose or duplicate landed work.

### D8 — The term `move note`

The note of D2 is called the move note, never a hand-back, since the shed driver already calls every non-`done` stop a hand-back.
The glossary gains the line:
`- **move note**: what a cancelled run's writer learned, written for the fresh run that replaces it. Never: hand-back, handoff.`

Rationale: the entry's word "hand-back" collides with the driver's stop report and the tackler's `hand-back` verdict.

## Constraints

- `lym batten cancel` sets a run-form status back to `draft`, which frees the board's run lock (`internal/boardengine/runlock.go`).
- With `--remove`, cancel archives each branch tip under `archive/<slug>/<12 hex of tip>` in its own repository, pushes the tags,
  and deletes both branches locally and on origin;
  its `archive_tags` output lists the code branch tip's tags only, and the records tip's tag is not reported (`internal/battencli/cancel.go`, `internal/battencli/cancelcmd.go`).
- Cancel without `--remove` keeps the pair and makes no archive tags, and the slug refuses to run again until `--remove` runs.
- A task that never ran, or whose pair was removed, records its parent only in the board entry's `project`;
  `_lym/fabric/origin.json` exists only while a pair exists.
- A task's owner machine is read through its parent (`internal/boardengine/owners.go`), so a moved task's fresh run starts on its new parent's owner;
  it needs nothing local to the old machine, since the archive tags are on origin and the note is on the board.
- Cancel refuses on a machine other than the entry's owner (`PATTERN-owner-machine`).
- An entry with a stored decision record runs from that record and never reads a body edit (`internal/boardengine/decision.go`).
- `PATTERN-stencil-ownership` and `PATTERN-heddle-instructions`: the procedure lives in the orch role text, and the glossary line in heddle's shipped glossary.
- No PATTERN entry changes: the move note is no run record a sole writer owns, since nothing commits it.

## Auto-mode assumptions

- Guidance alone, not a verb (D1), picked without the operator.
- The note's name and term, `move note` at `_lym/move-note.md` (D2, D8), picked without the operator.
- The note stays uncommitted, departing from the entry's archive-tag wording (D2), picked without the operator.
- The same entry carries on rather than a new task (D5), picked without the operator.
- Read-only use of the old commits (D6), picked without the operator.

## Open risks

- Re-running a slug after `lym batten cancel --remove` is refused today while the records repository keeps a stale tracking ref of the deleted records branch;
  entry `stale-records-ref-blocks-rerun` (running) fixes that.
  Until it lands, D5's re-run of the same slug needs that entry's workaround, a pruning fetch of the records repository.

## Acceptance criteria

- Review: `contracts/orchrole/role-beams.md` holds a section on moving a running task to another parent that states guidance alone with existing verbs and no recipe step (D1).
- Review: that section has the orch pause the run, ask the live writer by creel for `_lym/move-note.md` with the four parts, and copy the note to its `.scratch/` before the cancel (D2).
- Review: that section has the orch write the move note itself when none of the writers is live (D3).
- Review: that section gives the order pause and note, `lym batten cancel <slug> --remove` from the prime, the move note section in the body or, for an entry with a stored record, in the record through `--record-file`, the `project` change, and release by the new parent's orch (D4).
- Review: that section keeps the same entry and files no new task (D5).
- Review: that section forbids cherry-picking, merging or checking out the archive tag and has the fresh run read the diff from the merge base (D6).
- Review: that section stops the move when cancel refuses, lifting the pause, and excludes beam projects, grouping projects and front tasks (D7).
- Review: `internal/heddleengine/glossary.md` holds the `move note` line of D8.
- Review: `contracts/orchrole/orch-role.md`'s run loop points to the new section beside the cancel lines, and its line on `role-beams.md` names moving a running task (D1).
