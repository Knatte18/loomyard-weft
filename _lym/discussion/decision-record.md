# Decision record — turn-batching

## Goal

Cut the turns of the heavy roles by telling each to request independent calls together, make independent edits together and run its gate once after the last edit, through one sentence per role stencil and one sentence shuttle appends to every Claude session's system prompt.

## Scope

- In: the batching sentence in eight stencils under `contracts/stencils/`: `darn/darn-template-write.md`, `webster/webster-body-implementer.md`, `loom/loom-template-discussion.md`, `loom/loom-template-plan.md`, `twill/twill-directive.md`, `burler/burler-step-1-explore.md`, `burler/burler-template-fixer.md` and `landing/landing-template-fixer.md`.
- In: one batching sentence in the appended system prompt that `internal/shuttleengine/claudeengine` writes to the run directory's `append-system-prompt.txt`, with its tests and the package doc.
- Out: a `PostToolUse` hook that nudges toward batching.
- Out: a bench before the change; the next runs are measured by re-running the turn audit's scripts.
- Out: handing an agent pre-read or digested source.

## Decisions

### D1 — Stencil wording says "message", not "turn"

Each stencil sentence follows the report's section 5 wording for its role, with "turn" replaced by "message", the unit the board entry's rule names.
Rationale: one message carries several tool calls, and the Edit tool applies several edits when they arrive in one message; a turn can span several messages, so "turn" does not ask for the batching.

### D2 — Placement per role

- Darn writer: Step 4's edit and test items carry the gather, edit and gate sentence.
- Webster implementer: step 2 of the per-card sequence, beside the `lym quarry resolve` read, carries it per card.
- Plan and Discussion writers: their explore step carries "Issue every search and read you can name in one message; wait for a result only to choose the next call."
- Twill: the fork boilerplate, which every lens fork copies, carries the same sentence for its further calls.
- Burler review: the explore step carries "Read the files of your focus in one message, then check them," worded to cover a solo round's target too.
- Burler fixer: its sequence's fix step carries "Read the findings' files together; apply the corrections to one file in one message."
- Landing fixer: its `## What to read` section carries the Plan writer's sentence.

Rationale: each sentence sits at the step where the role first reads or edits, so it is read at the moment it applies.
The burler stencils serve the Discussion, Plan and Code rounds alike, so one sentence each covers every burler row of the report.

### D3 — The darn gate runs once per commit, after its last edit

The darn writer's "test as you work" becomes one `lym gate test --wait` after the last edit of each commit.
Rationale: the stencil also asks for small commits that each leave the tree green, so the gate runs once per commit rather than once per session; within a commit it runs once.
Bound: a commit's edits go unchecked until its gate, which is the same bound the commit rule already sets.

### D4 — Webster's card gate stays per card

The Webster sentence runs the card's gate once, after that card's last edit.
Rationale: the card gate and its commit per card are the plan's resume trail and stay mandatory; the sentence removes repeated gate runs within a card only.

### D5 — The appended system prompt carries the batching sentence after the creel sentence

The engine appends the sentence "Request every call whose arguments you already know in one message, and read what you need before you edit; wait for a result only to choose the next call." after the creel trust sentence, separated by a space, in the one appended prompt.
The deny notice keeps its place before both.
The package's init check refuses a line break in the sentence as it does for the creel sentence, since the file is one line.
Rationale: the board entry places it beside the creel sentence in one appended prompt; the engine already delivers that prompt through `--append-system-prompt-file` on the launch and resume lines, so the sentence survives a resume.
Bound: a session opened with `lym shuttle open` writes no run directory and carries neither sentence, as it carries no creel sentence today.

## Constraints

- `PATTERN-stencil-ownership`: the sentences live in the stencils, read at call time; no Go code embeds them.
- `PATTERN-shuttle-provider-seam`: the appended-prompt sentence stays inside `internal/shuttleengine/claudeengine`.
- Stencil markers keep their positions: `contracts/stencils/markers_test.go` pins the twill marker inside the Plan writer's Step 2, and the burler explore step's `fork_reading` marker stays on the `tool_use_rules` line.
- Markdown keeps one sentence per line.

## Auto-mode assumptions

- The report's section 5 wording is the agreed text; only "turn" changes to "message".
- Code-burler rows need no separate stencil, since the burler stencils are shared across rounds.

## Open risks

- The sentences may cut fewer turns than the report's ceiling; the later audit measures it.
- A merged message delivers its results together, which can lengthen a later context slightly.

## Acceptance criteria

- D1 and D2: each of the eight stencils carries its sentence at the named step, worded with "message", and the stencil marker tests stay green.
- D3: the darn stencil's Step 4 asks for one `lym gate test --wait` after the last edit of each commit, and keeps the green-commit rule.
- D4: the Webster stencil keeps the per-card gate and commit, and asks for the card's gate once after its last edit.
- D5: a prepared run's `append-system-prompt.txt` holds the deny notice, the creel trust sentence and the batching sentence, in that order and space-separated, or the two sentences alone with no deny; the launch and resume lines both name the file; the package doc describes the file's contents.
