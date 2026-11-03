# The decision record on the board, and splitting a discussed task

`discussed-task-split` · Note · feature, loom, board, batten, workspaces

## Seen

A task's decision record lived only in its run's weft (`_lym/discussion/decision-record.md`): a cancelled run left it in the records history, and no new task took it along.
So when Discussion finds a task too large, or the operator decides a tapestry run should rather be split, what Discussion found is lost or must be copied out by hand.
Several notes circled the same question (where the decision record lives, how a task is split); this note joins them.

## Core: the decision record on the board (operator, 2026-11-01)

- When Discussion is approved, after its review, the decision record is copied into the task's board entry, and the entry is flagged as discussed.
- A task flagged discussed skips Discussion: tapestry starts at Plan from the record, and darn takes the record as final, as darn takes its task text as already discussed today.
- Changing one's mind loses nothing: the run is cancelled, the record stays on the board, and it can be split into discussed tasks or made a project.
- lym itself writes the record field, at Discussion's approval and at each `lym loom decision add`, so the field is exempt from the lock on a claimed entry.
  That lock exists only so the orch does not change a task's text after its agents have read it; a record written by lym is not that.
- Built (2026-11-03): task `decision-record-on-board` stores the record and the flag.
  The record is the file `decision-<slug>.md` at the Bolt root beside `board.json`, which carries only a `discussed` boolean; `lym board get` shows the flag and `--record` prints the record; `lym loom decision store` re-copies it by hand.
  Runs already past Discussion are not backfilled, and approval inside a sub-run block stores no record until the gate follow-up.
- Built (2026-11-03): task `shed-sub-run-block` adds the block row, the recipe `motif` (tapestry's Discussion rows) and a trial recipe `arras` (`motif` through a block, then tapestry's rows from Drift-Baseline on).
- Still open: skipping Discussion for a discussed entry, the gate check on that block, now the running task `tapestry-motif-block`.

## Discussion as a run of its own (operator, 2026-11-03)

How a task is best cut into smaller tasks is known only once its Discussion is done, so the split comes after Discussion, not before.

- A Discussion-only run, the recipe `motif` (operator, 2026-11-03: the motif is the design decided before the weaving; `cartoon`, the drawing the weaver follows, is kept for a later Plan block), interactive or not, runs Discussion-Write and its review and nothing else, then stores the record and flags the entry discussed, as the core above says.
  The recipe exists; the running task `tapestry-motif-block` has tapestry run it as a block, removes `arras`, and leaves `motif` runnable alone.
- A discussed entry then goes one of two ways, chosen after its Discussion:
  1. Tapestry from Plan, skipping Discussion-Write: Plan-Write and Webster as today.
  2. The part split: the record's scope is cut into darn tasks with contracts and `depends_on`, which run independently, each a task of its own.
- Open: whether Discussion-Write also writes the part split, since it has the overview, or a separate step does it only when way 2 is chosen, so a task going way 1 pays nothing for it.
  The orch's recommendation: Discussion-Write only judges in a line or two whether the scope has clear seams, and the split itself is a separate step on the discussed entry, much as `Parts-Write` is today.

## Ways to split, simplest first

1. The operator, or the board agent of note `board-agent-button`, splits a discussed entry into tasks with `depends_on`, each with its part of the record and flagged discussed.
   The same agent can split an undiscussed entry before it runs, but then each part runs its own Discussion and rereads much of the same code; a tapestry part carries about $23 of fixed cost, so this pays best when the parts are darn-sized.
2. Discussion finds the task too large or proposes a split: it writes its findings and the proposed parts into the record, tells its parent and ends its turn.
   It files no task or project and starts nothing; the operator decides, and the parts become plain tasks in the prime or, for a large one, a project and beam spawned from the parent, never from the old task's worktree.
   Its prompt must hold back: one part is the default, a split only where the seams are clear and each part pays for its own Plan and review, and small parts go to darn.
3. `stitch`: a stand-alone loom recipe for a large task's whole course, chosen before the run starts.
   Preflight, Discussion, a Pre-Plan producer that splits the decided scope into parts with contracts (what each takes in, delivers and exports, and which parts it depends on), a row with no agent that turns the entry into a project and files each part as a darn task, then an inner pawl run through `battenshed.NewInnerRun` that brings up the beam, waits until every part has landed and lands the beam.
   Parts run with review off; one review covers the whole diff after the last lands.
   It is tried first as a hand-run test project with the projects and beams that exist, measured against tapestry runs of the same size (`docs/research/token-audit-2026-10-22`), and built only if it saves tokens.
   The risk: tapestry's build is cheap because Webster runs on Sonnet following an Opus plan, while parts under a coarse pre-plan likely need Opus writers.

## Open

- A split record's format: each part must still pass the discussion check.
- Whether the old run's commits, if any, are kept when a run is cancelled for a split.
- `stitch`: contract precision, how large the whole-task review's fixes may be, and how often the choice between `stitch` and tapestry before the run is right.
- Settled (2026-11-03): the records live in a file of their own beside `board.json`, not in it.
