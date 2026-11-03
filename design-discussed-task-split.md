# The decision record on the board, and splitting a discussed task

`discussed-task-split` · Note · feature, loom, board, batten, workspaces

## Seen

A task's decision record lived only in its run's weft (`_lym/discussion/decision-record.md`): a cancelled run left it in the records history, and no new task took it along.
So when Discussion finds a task too large, or the operator decides a tapestry run should rather be split, what Discussion found is lost or must be copied out by hand.
Several notes circled the same question (where the decision record lives, how a task is split); this note joins them.

## Names (operator, 2026-11-03)

A board idea is a note; the decision record that Discussion, or the recipe `motif`, makes is a Sketch, written with a capital S as a proper name, so it does not read as a loose draft; a finished plan may later be a cartoon.
Code still says "decision record": nothing in code is renamed to Sketch.

## Core: the decision record on the board (operator, 2026-11-01)

- When Discussion is approved, after its review, the decision record is copied into the task's board entry, and the entry is flagged as discussed.
- A task flagged discussed skips Discussion: tapestry starts at Plan from the record, and darn takes the record as final, as darn takes its task text as already discussed today.
- Changing one's mind loses nothing: the run is cancelled, the record stays on the board, and it can be split into discussed tasks or made a project.
- lym itself writes the record field, at Discussion's approval and at each `lym loom decision add`, so the field is exempt from the lock on a claimed entry.
  That lock exists only so the orch does not change a task's text after its agents have read it; a record written by lym is not that.
- Built (2026-11-03): task `decision-record-on-board` stores the record and the flag.
  The record is the file `decision-<slug>.md` at the Bolt root beside `board.json`, which carries only a `discussed` boolean; `lym board get` shows the flag and `--record` prints the record; `lym loom decision store` re-copies it by hand.
  Runs already past Discussion are not backfilled.
- Built (2026-11-03): task `readme-shows-discussed` (`1bbcc3b1`).
  The board README and every project file link a discussed entry's record as `Sketch` under its slug, Done entries included.
- Built (2026-11-03): task `shed-sub-run-block` adds the block row and the recipe `motif` (tapestry's Discussion rows).
- Built (2026-11-03): task `tapestry-motif-block`.
  Tapestry defines Discussion once, in `motif-recipe.yaml`, and runs it through a block row named `Motif`.
  The block's gate `undiscussed` passes the block over when the entry is flagged discussed: the stored record is seeded as the run's decision record and the run goes on at Drift-Baseline, so Plan-Write and the reviews read it as they read an approved one.
  Approval inside the block stores the record.
  The trial recipe `arras` is removed.
- Still open: darn taking the stored record as final for a discussed entry.

## Discussion as a run of its own (operator, 2026-11-03)

How a task is best cut into smaller tasks is known only once its Discussion is done, so the split comes after Discussion, not before.

- A Discussion-only run, the recipe `motif` (operator, 2026-11-03: the motif is the design decided before the weaving; `cartoon`, the drawing the weaver follows, is kept for a later Plan block), interactive or not, runs Discussion-Write and its review and nothing else, then stores the record and flags the entry discussed, as the core above says.
  Built (2026-11-03): a `motif` task runs alone, lands no code and returns the entry to `draft`; landing only prunes the board.
- A discussed entry then goes one of two ways, chosen after its Discussion:
  1. Tapestry from Plan, skipping Discussion-Write: Plan-Write and Webster as today. Built, see above.
  2. The part split: the record's scope is cut into darn tasks with contracts and `depends_on`, which run independently, each a task of its own.

## The split is a cold agent of its own (operator, 2026-11-03)

- The Sketch stays as it is: Discussion-Write writes no split and judges no seams, so a task going way 1 pays nothing for the split.
- A separate cold agent splits a discussed entry when way 2 is chosen: it reads the stored Sketch and the code the Sketch names, and proposes the parts.
- The proposal holds, per part: the Sketch decisions it builds, the packages it changes, its contract (what it takes in, delivers and exports), its `depends_on`, the Open risks and Acceptance criteria it carries, and a recipe and profile.
  It checks that no two parallel parts change the same package.
- The agent is a role of its own so it can be tuned hard: its prompt, model and effort are set apart from Discussion-Write's.
- Its proposal goes to the operator, who approves it before the entry becomes a project and its parts become tasks.
- Seen (2026-11-03): the first split, F18 `codex-engine` into project P2, was done by the orch from the Sketch's headings in a few calls, without checking packages, contracts or criteria per part; that shallow split is what this agent replaces.

## Ways to split, simplest first

1. The operator, or the board agent of note `board-agent-button`, splits a discussed entry into tasks with `depends_on`, each with its part of the record and flagged discussed.
   The same agent can split an undiscussed entry before it runs, but then each part runs its own Discussion and rereads much of the same code; a tapestry part carries about $23 of fixed cost, so this pays best when the parts are darn-sized.
2. `stitch`: a stand-alone loom recipe for a large task's whole course, chosen before the run starts.
   Preflight, Discussion, a Pre-Plan producer that splits the decided scope into parts with contracts (what each takes in, delivers and exports, and which parts it depends on), a row with no agent that turns the entry into a project and files each part as a darn task, then an inner pawl run through `battenshed.NewInnerRun` that brings up the beam, waits until every part has landed and lands the beam.
   Parts run with review off; one review covers the whole diff after the last lands.
   It is tried first as a hand-run test project with the projects and beams that exist, measured against tapestry runs of the same size (`docs/research/token-audit-2026-10-22`), and built only if it saves tokens.
   The risk: tapestry's build is cheap because Webster runs on Sonnet following an Opus plan, while parts under a coarse pre-plan likely need Opus writers.

## Open

- A split record's format: each part must still pass the discussion check.
- Whether the old run's commits, if any, are kept when a run is cancelled for a split.
- `stitch`: contract precision, how large the whole-task review's fixes may be, and how often the choice between `stitch` and tapestry before the run is right.
- Settled (2026-11-03): the records live in a file of their own beside `board.json`, not in it.
