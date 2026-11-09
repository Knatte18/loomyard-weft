# The decision record on the board, and splitting a discussed task

`discussed-task-split` · Note · feature, loom, board, batten, workspaces

## Seen

A task's decision record lived only in its run's weft (`_lym/discussion/decision-record.md`): a cancelled run left it in the records history, and no new task took it along.
So when Discussion finds a task too large, or the operator decides a tapestry run should rather be split, what Discussion found is lost or must be copied out by hand.
Several notes circled the same question (where the decision record lives, how a task is split); this note joins them.

## Names (operator, 2026-11-03)

A board idea is a note; the decision record that Discussion, or the recipe `motif`, makes is a Sketch, written with a capital S as a proper name, so it does not read as a loose draft; a finished plan may later be a cartoon.
Code still says "decision record": nothing in code is renamed to Sketch, except the board type below.
Since 2026-11-05 an entry that holds a motif run's record is also a Sketch on the board: a note of the built-in type `sketch`.

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
- Built (2026-11-07, `c5e44a93`): the design page of every discussed entry also carries a `Decision record: [Sketch](decision-<slug>.md)` line under its metadata line, so a reader who opens the page from the board reaches the record.
  `RenderToDisk` renders a discussed entry whose record file is missing as undiscussed, so no page links a missing record.
- Built (2026-11-03): task `shed-sub-run-block` adds the block row and the recipe `motif` (tapestry's Discussion rows).
- Built (2026-11-03): task `tapestry-motif-block`.
  Tapestry defines Discussion once, in `motif-recipe.yaml`, and runs it through a block row named `Motif`.
  The block's gate `undiscussed` passes the block over when the entry is flagged discussed: the stored record is seeded as the run's decision record and the run goes on at Drift-Baseline, so Plan-Write and the reviews read it as they read an approved one.
  Approval inside the block stores the record.
  The trial recipe `arras` of that time was removed; the name now belongs to the recipe built on 2026-11-09, see below.
- Built (2026-11-05): task `raddle-recipe` (`257ab79a`).
  Record-Write copies a discussed entry's stored record into the run's decision record for every recipe with that row, so a darn entry flagged `discussed` now builds from its stored record.
  That closes the former open point that darn take the stored record as final.

## Discussion as a run of its own (operator, 2026-11-03)

How a task is best cut into smaller tasks is known only once its Discussion is done, so the split comes after Discussion, not before.

- A Discussion-only run, the recipe `motif` (operator, 2026-11-03: the motif is the design decided before the weaving; `cartoon`, the drawing the weaver follows, is kept for a later Plan block), interactive or not, runs Discussion-Write and its review and nothing else, then stores the record and flags the entry discussed, as the core above says.
  Built (2026-11-03): a `motif` task runs alone and lands no code; landing only prunes the board.
  Built (2026-11-05): task `sketch-board-type` (`f2ae537d`).
  A finished motif run no longer leaves the task `discussed`: the entry becomes a Sketch, a note of the built-in type `sketch` (handle letter `S`) that keeps its module labels and loses its status and the fields only a run, a landing or the dependency graph read (`depends_on`, `isolated`, `recipe`, `profile`, `run_settings`, `pull_request`, `notify_on_land`).
  A Sketch is not a task and never runs.
  The board lists Sketches last in `lym board list --text` and first among the Notes in the README; the `Discussed` README layer is gone, and every board write turns an older open `discussed` task into a Sketch.
  A Sketch another entry depends on is refused: the entry stays `draft` with a warning.
  Built (2026-11-03): task `beam-after-motif` (`cfccb80a`).
  When the entry is then made a project, `lym beam create` removes the slug's done motif run, once its pair is gone, instead of refusing with `already completed`; the envelope reports it as `motif_run_cleared`.
  Seed-Child records the child recipe in batten's run directory, so a done run still tells it ran `motif`; a done motif run seeded before that has no record and still needs `lym batten cleanup --apply`.
- A discussed entry then goes one of two ways, chosen after its Discussion:
  1. Tapestry from Plan, skipping Discussion-Write: Plan-Write and Webster as today. Built, see above.
  2. The part split: the record's scope is cut into darn tasks with contracts and `depends_on`, which run independently, each a task of its own.

## Triage of a Sketch (operator, 2026-11-05)

Built (2026-11-05): task `raddle-recipe` (`257ab79a`), the recipe `raddle` (the raddle spreads the warp into sections before it is beamed).
It is a record-only loom recipe with the rows Record-Write, Raddle-Write and Raddle-File, run through batten like `motif`: a worktree of its own, a pane, a board status and notices, no code landed.
Built (2026-11-05): task `sketch-promote-raddle` (`867b4eeb`).
`lym board promote <slug>` of a Sketch, or `lym board promote S3` by handle, starts it with recipe `raddle`; `--recipe raddle` is still accepted, and any other recipe on a Sketch is refused, naming the plain promote.
A Sketch with no stored record is refused, naming `motif`.

- Raddle-Write is a cold Opus agent (loom key `raddle`, default `opus[high]`) that reads the Sketch, the board and a bounded read of code, and writes a proposal: one tapestry from Plan (way 1 above), several darn tasks (way 2), a mix, a project, or a beam, with the order as `depends_on`.
  It designs nothing; the Sketch has done that.
- Interactive by default: the operator approves the proposal in the agent's pane (`lym loom raddle-approve`), may question it and have it revise first.
  The loom key `raddle_auto_approve` files with no operator input; it stays off until the splits are tuned well enough.
- Built (2026-11-09, `2ad39d26`): task `raddle-perch-mode`.
  A raddle run whose recipe rows hold a split perch (a Bouncer row whose `artifact_paths` name the raddle proposal) records a third mode, `perch`, decided by the rows and never by the recipe's name.
  The perch's approval files the proposal in place of the operator's, and `raddle_auto_approve` is never read; `raddle-approve` refuses, naming the perch.
  Perch mode allows one task (shape `tapestry`, or shape `tasks` with one part) or one project (shape `project` whose slug is the run's own entry); shape `beam`, several task parts and a project under another slug fail the `perch-outcome` check.
  A perch-mode `lym loom decision add` is refused from Raddle-File on, with a way forward.
  Filing a perch-mode project as the converted entry comes with `arras-family-filing`; here such a proposal only passes the checks.
- Built (2026-11-09, `df15bb6a`): task `arras-recipe-perch`.
  The recipe `arras` runs the `Motif` block (Discussion), `Raddle-Write`, a split perch and `Raddle-File` in one run that lands no code, so a task goes from its Discussion to filed parts with the split reviewed before it is filed.
  The perch is the rows `Raddle-Bouncer`, `Raddle-Burler-Review` and `Raddle-Burler-Fix`; the rubric stencil `loom-rubric-raddle-review.md` judges the split against the whole record, and the loom keys `split_*` tune it.
  `Raddle-File`'s `on_stuck` returns to `Raddle-Write`, and a fresh entry there re-arms a settled perch for a new cycle with the full budget.
  An arras run restarted after its Discussion approval skips Discussion and resumes at `Raddle-Write`.
  Not run end to end yet: an arras task through Discussion, the perch and `Raddle-File`, including one `Raddle-File` return.
  Open: a decision added after the perch's last approving round reaches every part's record but is not reviewed against the split's seams; accepted in the design.
- Raddle-File re-checks the board, then files every entry in one write, each with its part of the record stored and `discussed` set, creates the beam for shape `beam` and hands the Sketch slug and proposal to the beam's orch in Beam-Up.
  Built (2026-11-07, `731004c3`): filing releases nothing.
  Every filed task, project members included, stays `draft` until the operator releases it with `lym board set-status`; the earlier release step is removed.
  Built (2026-11-09, `8ce95432`): task `project-release-members`.
  `lym board set-status` with a null status on a project whose own status is empty releases, in the same write, every direct member task of that project in `draft`, so one command releases a filed family; the envelope's `notice` names the released members.
  Members already released, run-held or done, nested projects and their members stay as they are, and a null status on a project that holds a run status or `done` is refused.
  The proposal carries an optional `order_departures` list (`{parts, record_order, cost}`) naming each place its chain order departs from an order the Sketch states, and the interactive approval table shows a dependencies column, the departures and the draft notice.
  The stencil orders parts that share a package by the Sketch's own order first.
  The table, the departures and the draft notice are instructions to the agent, not enforced in code.
  A moved board, an invalid proposal or an approval-hash mismatch returns to Raddle-Write; the third return halts the run.
- The stencil carries the rule that a tapestry's fixed start cost often favours fewer, larger parts.
  The cost figures come from the hub's own runs: every agent session is recorded as `_lym/usage/<session>.json`, `lym report cost` gives the fixed-plus-per-size fit per recipe and profile, and `lym report raddle` compares each proposal's predicted cost with its landed parts, so the prompt can be tuned.
  A Codex session's dollar cost stays unknown, so its runs enter the report in hours alone.
- A raddle store beside `board.json` keeps each Sketch's proposals, record and filed slugs; `lym board raddle <slug>` prints a stored proposal.
- Not run end to end against a live hub yet: an interactive raddle run through approval and filing, and a `beam` shape run through Beam-Up's hand-off.

Still open here: the cold split agent below is now Raddle-Write, so its tuning follows from `lym report raddle` once runs have landed.

## The split is a cold agent of its own (operator, 2026-11-03)

Built as Raddle-Write above; what follows is the original design.

- The Sketch stays as it is: Discussion-Write writes no split and judges no seams, so a task going way 1 pays nothing for the split.
- A separate cold agent splits a discussed entry when way 2 is chosen: it reads the stored Sketch and the code the Sketch names, and proposes the parts.
- The proposal holds, per part: the Sketch decisions it builds, the packages it changes, its contract (what it takes in, delivers and exports), its `depends_on`, the Open risks and Acceptance criteria it carries, and a recipe and profile.
  It checks that no two parallel parts change the same package.
- The agent is a role of its own so it can be tuned hard: its prompt, model and effort are set apart from Discussion-Write's.
- Its proposal goes to the operator, who approves it before the entry becomes a project and its parts become tasks.
- Seen (2026-11-03): the first split, F18 `codex-engine` into project P2, was done by the orch from the Sketch's headings in a few calls, without checking packages, contracts or criteria per part; that shallow split is what this agent replaces.

## Ways to split, simplest first

1. The operator, or the board agent the prime's `board` button opens (task `board-agent-button`, `2def9684`, landed 2026-11-04; it loads `ly:board-policy` and `ly:board`), splits a discussed entry into tasks with `depends_on`, each with its part of the record and flagged discussed.
   The same agent can split an undiscussed entry before it runs, but then each part runs its own Discussion and rereads much of the same code; a tapestry part carries about $23 of fixed cost, so this pays best when the parts are darn-sized.
2. `stitch`: a stand-alone loom recipe for a large task's whole course, chosen before the run starts.
   Preflight, Discussion, a Pre-Plan producer that splits the decided scope into parts with contracts (what each takes in, delivers and exports, and which parts it depends on), a row with no agent that turns the entry into a project and files each part as a darn task, then an inner pawl run through `battenshed.NewInnerRun` that brings up the beam, waits until every part has landed and lands the beam.
   Parts run with review off; one review covers the whole diff after the last lands.
   It is tried first as a hand-run test project with the projects and beams that exist, measured against tapestry runs of the same size (`docs/research/tokens/token-audit-2026-10-22`), and built only if it saves tokens.
   The risk: tapestry's build is cheap because Webster runs on Sonnet following an Opus plan, while parts under a coarse pre-plan likely need Opus writers.
   Since 2026-11-09 the recipe `arras` covers Discussion and a reviewed split (filing a project comes with `arras-family-filing`), so `stitch` would add only the inner run that lands the whole family under one review.

## Open

- A split record's format: each part must still pass the discussion check.
- Whether the old run's commits, if any, are kept when a run is cancelled for a split.
- `stitch`: contract precision, how large the whole-task review's fixes may be, and how often the choice between `stitch` and tapestry before the run is right.
