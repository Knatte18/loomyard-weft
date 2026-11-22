# Orch procedure to move a running task to another parent

The orch gets one written procedure for moving a running task to another parent.
The procedure cancels the old run and starts a fresh run under the new parent, with a move note that carries over what the old run learned.
It uses only verbs that exist today and adds no batten verb, flag or recipe row.

## What changed

- `contracts/orchrole/role-beams.md` gains the section "Moving a running task to another parent", in five steps:
  1. The run's live writer (the Discussion writer, the plan writer or the Webster master) is asked by creel, before any pause, for the note's first three parts:
     what is done, the decisions taken and what is left.
     The orch writes the note in its own `.scratch/` and holds the run with `lym tender pause <slug>`.
     When no writer is live, or the writer refuses, the orch holds the run and writes the whole note itself.
     It records the tip SHA and merge base after the hold, and never holds the run with `lym batten pause`, since the inner run ignores that pause.
  2. `lym board get` checks that the new parent can take a member.
     A beam also has to lack a ready record (`_lym/landing/beam-ready.json` on its `-weft` branch).
     When that record cannot be read for a beam on another machine, the orch asks the beam's orch, or the operator.
     A failed check stops the move before the cancel.
  3. The orch runs `lym batten cancel <slug> --remove` from the prime.
     It tells a partial finish from a refusal by the way forward the output names, and never runs `lym tender resume` without a slug.
     It names the code archive tag from the recorded tip.
     Before writing to the board, it confirms that the tag is on origin and the slug's records branch is gone.
     After ten minutes it pushes the tag by hand, and if the records branch is still listed it stops.
  4. One upsert of the same entry writes the move note into what the fresh run reads and sets `project` in the same call.
     The note goes in a `## Move note` body section, or for a discussed entry in a `### Move note` under the record's `## Notes for the plan writer` through `--record-file`.
     It names the tag fetch the fresh run makes.
     When the board refuses the `project` value, the upsert runs again without it, which keeps the note.
  5. The orch whose worktree is the new parent releases the entry.
     While `stale-records-ref-blocks-rerun` has not landed, it first runs a pruning fetch of the records repository.
- The fresh run reads the diff from the merge base to the archive tag and never cherry-picks, merges or checks it out.
  Beam projects, grouping projects and front tasks are never moved this way, and the moved work stays on the same entry and slug.
- `contracts/orchrole/orch-role.md` points to the section from its list of role files and from run-loop item 3, beside the cancel lines.
- `internal/heddleengine/glossary.md` gains the term **move note**, ruling out "hand-back" and "handoff".
  The shed driver already uses "hand-back" for its stop report.

## Design decisions

- The move note lives in the orch's `.scratch/` until the board write, and is never committed or written in the task worktree.
  So no archive tag holds it; the board copy is the durable one, as the decision record's D2 sets.
- This hub's darn config asks for a report under `docs/research/`, but this task asks for no report and finds nothing to report.
  So the run commits none.

## Merge danger

**Door:** two-way
**Blast radius:** orch
