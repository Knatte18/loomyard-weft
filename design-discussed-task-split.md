# The decision record on the board, and splitting a discussed task

`discussed-task-split` · Note · feature, loom, board, batten, workspaces

## Seen

A task's decision record lives only in its run's weft (`_lym/discussion/decision-record.md`): a cancelled run leaves it in the records history, and no new task takes it along.
So when Discussion finds a task too large, or the operator decides a tapestry run should rather be split, what Discussion found is lost or must be copied out by hand.
Several notes circled the same question (where the decision record lives, how a task is split); this note joins them.

## Core: the decision record on the board (operator, 2026-11-01)

- When Discussion is approved, after its review, the decision record is copied into the task's board entry, and the entry is flagged as discussed.
- A task flagged discussed skips Discussion: tapestry starts at Plan from the record, and darn takes the record as final, as darn takes its task text as already discussed today.
- Changing one's mind loses nothing: the run is cancelled, the record stays on the board, and it can be split into discussed tasks or made a project.
- lym itself writes the record field, at Discussion's approval and at each `lym loom decision add`, so the field is exempt from the lock on a claimed entry.
  That lock exists only so the orch does not change a task's text after its agents have read it; a record written by lym is not that.

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

## Evidence from the plan-split benches (2026-11-01 and 2026-11-02)

The plan-split writer (`Parts-Write` cuts the scope into parts with contracts, part writers plan each part, a chair joins them) is the planning half of `stitch` (`docs/research/plan-split-tune-2026-11-02.md`).
The operator's aim (2026-11-02): parts are not joined into one plan in the long run but implemented separately, which is where time and tokens may be saved.

- Both inputs split into 6 parts; the joined plans had 36 and 37 cards against 20 in the solo plans that landed, for the same code locations.
  About 85% of the extra cards were changes the solo plan had bundled into too-large cards; the rest were per-part overhead (PATTERN and doc cards per part, a card for one registry line), and about 9 cards were too small to be worth a card.
- Planning cost and wall time were close to the solo planner's after the tuning ($18.69 against $14.93, 24 against 25 minutes; $29.36 against $28.03, 29 against 32 minutes), from one run each.
- How well a task splits decides the gain: `warp-visibility` had a critical path of 8 of 36 cards and 15 cards on its widest level, while `run-verbs-shared` had a path of 20 of 37 cards and 119 dependencies, so its parts are one chain and gain nothing from separate implementation.
  The split writer should be able to answer that a task does not split, and leave it to the solo planner.
