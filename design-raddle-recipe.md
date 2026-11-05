# Recipe raddle turns a Sketch into tasks or a project

`raddle-recipe` · Task · feature, loom, board, batten · running · Finalize

## Wanted (operator, 2026-11-05)

A recipe `raddle` (the raddle spreads the warp into sections before it is beamed) runs on a Sketch, a note of type `sketch`, and decides what it becomes:

- one tapestry that starts at Plan from the stored record, or several darn tasks, or a mix;
- the order of the tasks, as `depends_on`;
- whether the whole becomes a project, or a beam.

Only this run turns a Sketch into tasks; a Sketch never runs itself.

## The agent

- A role of its own, cold, on Opus, so its prompt, model and effort are tuned apart from Discussion-Write's.
- It reads the Sketch, the board (entries that exist or run in the same packages) and only enough code to place each part: the packages and files the Sketch names, which packages each part changes, roughly how much, and who calls whom for the order.
  It designs nothing; the Sketch has done that.
- Per part, the proposal holds: the Sketch decisions it builds, the packages it changes, its contract (what it takes in, delivers and exports), its `depends_on`, the Open risks and Acceptance criteria it carries, and a recipe and profile.
  It checks that no two parallel parts change the same package.
- It may answer "one tapestry", which costs no split.

## Cost

- A tapestry carries a large fixed start cost, so one large tapestry beside two small darns is often worse than fewer, larger darns or one tapestry for the whole.
- The agent is given each recipe and profile's measured fixed and per-size cost from the hub's own run records (`lym report timing` and the token audits), not a figure written into its prompt.
- Its proposal states the expected cost of the split it proposes against the next-best one.
- Its choices are checked after landing against what the runs cost, so its prompt can be tuned.

## Approval

- The agent asks the operator nothing while it works: it writes a complete proposal first.
- Interactive by default: only then does the agent put its proposal to the operator and wait for their input; the operator may question it, correct the split, order or recipes, and the agent revises until the operator approves.
  This works like Discussion-Write's interactive mode.
- An auto-approve flag lets the run file its proposal with no operator input.
  It stays off until the agent's splits are tuned well enough to run alone; the operator's corrections in the interactive runs are what tune it.
- On approval the run files the tasks, project or beam; nothing is filed before.
  The board can move meanwhile, so the proposal is checked against the board again at approval.
- A proposal that becomes a project or beam hands the Sketch and the proposal to that beam's orch.

## Run

- It runs through batten like `motif`, in a worktree of its own forked from the parent, so the code it reads stays still while it reads; it lands no code, and its run shows on the board with a pane and notices.
- The glossary `docs/research/weaving-glossary.md` gains `raddle` as taken.
