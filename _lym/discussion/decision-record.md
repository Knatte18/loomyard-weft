## Goal

A loom recipe `raddle` runs on a Sketch and turns it into what the operator approves: one tapestry that starts at Plan, several tasks (darn, tapestry or a mix) ordered by `depends_on`, a project, or a beam.
A cold Opus agent proposes the split from the Sketch, the board, a bounded read of code and the hub's own measured run costs.
The operator approves the proposal in the agent's pane unless auto-approve is on, and only then does Go file it on the board.
Every run's agent cost is recorded so the proposal's cost figures come from the hub's own runs, and each proposal's prediction is later compared with what its tasks cost.

## Scope

In: D1 to D15 — the recipe and its start, Record-Write's seeding, the agent, the proposal and its gate, approval and the board re-check, filing and the raddle store, the beam hand-off, usage records and the two reports, the glossary row.

Out:

- Any review ring on the proposal: the operator's approval is its review.
- Splitting an undiscussed entry, and the `stitch` recipe of note `discussed-task-split`.
- `plan_after` (note `plan-after-dependency`) and moving running tasks (note `reparent-running-task`).
- Filing outside the Sketch's own parent level (note `beam-orch-root-project`).
- Backfilling usage for runs that ran before this binary.

## Decisions

### D1 — `raddle` is a record-only loom recipe of three rows

`contracts/recipes/raddle-recipe.yaml` (new) declares `contract: loom` in its record-only form, with rows Record-Write, Raddle-Write and Raddle-File and no build, review or Finalize row.
Raddle-Write is a new agent engine, `RaddleWrite`, and Raddle-File a new Go engine, `RaddleFile`, each one `Constructor` in `internal/shedrecipe`'s registry.
The recipe joins the closed recipe-name list, the embedded recipes and batten's child recipes, so batten runs it like `motif`: in a worktree of its own forked from the parent, with a pane, a board status and notices, landing no code.
Rationale: record-only form is the loom contract `motif` already uses, and Record-Write gives the run the Sketch as its decision record, which the agent and the part splice both read.

### D2 — A Sketch starts a raddle run only by promotion with recipe `raddle`

`lym board promote` gains an optional recipe, set in the same write as the kind.
A write that makes a Sketch a task must set recipe `raddle` and keep the `sketch` label, the stored record and the `discussed` flag; any other such write is refused, naming `lym board promote <slug> --recipe raddle` as the way forward.
The board admits the `sketch` label on a task only while its recipe is `raddle`, and such a task is a raddle run's entry, not a Sketch, to `boardengine.IsSketch` and the README.
The promoted entry is released, so its parent's dobby starts the run.
Batten's start guard refuses a raddle run on an entry with no stored record, naming a `motif` run as the way forward.
Bound: the rule admits exactly one recipe, so only a raddle run turns a Sketch into tasks.

### D3 — Record-Write seeds a discussed entry's stored record

For an entry flagged `discussed`, Record-Write copies the stored record to the run's decision record and writes the support log, instead of rendering the record from the entry's body.
This holds for every recipe with a Record-Write row: darn takes a discussed entry's record as final, and a pawl run takes a raddle-filed project's record, the Sketch, as its beam's project record.
Rationale: raddle files every part and project discussed (D9), and each must build from the record it was given.

### D4 — The raddle agent is a cold role of its own

Raddle-Write spawns one agent with stencil `contracts/stencils/raddle/raddle-template-agent.md` (new) and the loom config key `raddle`, default `opus[high]`, set apart from the loom key `discussion`.
It reads the run's decision record, `lym board list` and the entries touching the Sketch's packages or module labels, `lym loom peers`, `lym report cost` and `lym report timing`, and only the code needed to place each part: the packages and files the Sketch names, which packages each part changes, roughly how much, and who calls whom.
It designs nothing and asks the operator nothing until its proposal has passed the checks of D6.
The stencil points at the proposal spec and restates none of it; it carries the rule that a tapestry's large fixed start cost often makes fewer, larger parts or one tapestry cheaper, and no cost figure.

### D5 — The proposal is one YAML file with a spec and a sole parser

The agent writes `_lym/raddle/proposal.yaml`, in the format `contracts/specs/raddle-proposal-spec.md` (new) pins, parsed only by `internal/raddleparser` (new), which imports the standard library and `internal/discussionparser` only.
It holds:

- `shape`: `tapestry`, `tasks`, `project` or `beam`, and for `project` and `beam` the project's slug and title.
- `board_checked`: each board entry the agent weighed, with its `revised` stamp.
- `parts`: per part, its slug, title, type label, recipe (`tapestry` or `darn`), profile, the Sketch decision ids it builds, the packages it changes with a rough size, its contract (takes in, delivers, exports), its `depends_on`, the Open risks and Acceptance criteria it carries, the PATTERN changes it carries, and its expected cost.
- `cost`: the proposal's expected cost, the next-best split described in one line with its expected cost, and the `lym report cost` cells both rest on.

Shape `tapestry` has exactly one part, which builds every decision.
Every filed slug, part or project, is new and differs from the Sketch's, so no filed entry shares a slug with the Sketch's motif or raddle runs.

### D6 — Gate `raddle` checks the proposal

The checks:

- every Sketch decision is built by at least one part, and every Acceptance criterion and Open risk is carried by at least one part;
- each part's carried criteria name every decision id it builds, so each part record passes the discussion check;
- `depends_on` is acyclic and names only parts or entries on the board;
- no two parts without a `depends_on` path between them list a common package;
- each recipe is `tapestry` or `darn`, each profile exists for its recipe, each type label is in the board's vocabulary and is not `sketch`, and each new slug is free;
- both cost lines are present.

In auto-approve mode the gate holds Raddle-Write's handoff, re-prompting the live session up to 3 attempts and then halting the run for its parent, like Discussion-Write's gate.
In interactive mode `lym loom raddle-propose` and `lym loom raddle-approve` run the same checks (D7).

### D7 — Interactive approval by default, auto-approve by a key

The loom config key `raddle_auto_approve`, default `false`, chooses the mode, and a run profile's raddle block may set it.

- On: the session is autonomous; it ends on the proposal file, the gate runs, and the row goes to Raddle-File with no approval.
- Off: the session is interactive and ends on the approval record.
  The agent runs `lym loom raddle-propose` once its proposal is complete; it runs D6's checks and returns their findings, and its first passing call in a run stores the proposal as the cold proposal, committed under `_lym/raddle/`, and posts a notice to the parent orch that a proposal waits for the operator.
  The agent then puts the proposal to the operator, and revises the file while the operator questions or corrects the split, order, recipes or profiles.
  On the operator's explicit approval it runs `lym loom raddle-approve`, which runs D6's checks and D8's re-check and on success records the approval with the proposal's hash; the session's end takes the row to Raddle-File.

Bound: both verbs refuse outside a run whose current row is Raddle-Write, and `raddle-approve` refuses before a cold proposal is stored, on an invalid proposal and on a moved board, each naming what to fix.
The verbs cannot tell the operator from the agent, so, as with `lym loom sampler-done`, an approval with no operator input is prevented only by the stencil; the auto key is the sanctioned way to file without one.
Raddle-File files only a proposal whose hash matches the recorded approval when the key is off.
The cold proposal is stored once per run and never overwritten.

### D8 — The proposal is checked against the board again at approval and at filing

The re-check measures from the latest passing check (the last passing `raddle-propose`, or the gate pass in auto mode) and passes only when:

- every `board_checked` entry still on the board has the same `revised` stamp; one now done, or gone from the board, counts as landed and is named to the agent without failing the check;
- no task or project created after that point and absent from `board_checked` carries a module label the Sketch carries;
- every new slug is still free.

`lym loom raddle-approve` runs it, and Raddle-File runs it again inside its board write.
A failure in Raddle-File is Stuck back to Raddle-Write with the moved entries named, in a new session in the run's mode, so in interactive mode the revised proposal needs a new approval.
Raddle-File allows 2 such returns per run and then halts for the parent.
Raddle-File drops a `depends_on` on an entry gone from the board, which has landed.

### D9 — Raddle-File stores, then files in one board write, then creates the beam

Raddle-File is a Go row with no agent, and every step is skipped when its result is already there, so a retry after a crash finishes the steps it missed:

1. Store: under the raddle store (D10), it writes the approved proposal, the cold proposal and a copy of the Sketch's record.
2. Part records: a record `decision-<slug>.md` for each filed entry, spliced from the run's decision record by a new `internal/discussionparser` function.
   A part's record holds its Goal and Scope from its contract and packages, the decisions it builds copied whole with their ids, every `### Added after Discussion` entry, the Sketch's Constraints and Auto-mode assumptions, and the Open risks, Acceptance criteria and PATTERN changes it carries.
   A project's record, and the `tapestry` shape's one part's record, is the Sketch's record whole.
3. Board write, one `boardengine` write under the board lock, after D8's re-check:
   - `tapestry` and `tasks`: each part is filed as a discussed task at the Sketch's own project level.
   - `project` and `beam`: the project is filed at that level with no `sketch` label, discussed, and each part as a discussed task in it.
   - Filed tasks carry their recipe, profile, type label, the Sketch's module labels and `depends_on`, and a body naming the Sketch slug, the part's contract and packages; they are released, since the approval or the auto key is the operator's release.
   - The Sketch entry is marked done in the same write, which the run lock admits as a status change.
4. Beam: for `beam`, `lym beam create` for the new project, acting from the parent worktree.
5. Notice: one notice to the parent orch naming what was filed, recorded as sent in the row's state.

Bound: Raddle-File writes the board as the run's Go code with no strand name, so the board's refusal of a run agent creating a task still holds for the raddle agent's own `lym board` calls.
Since the Sketch entry is done, the run's later status writes skip it, its finish writes nothing to it, and batten's Land prunes it with its `decision-<slug>.md`; the raddle store keeps the copy.

### D10 — The raddle store on Bolt outlives prune

The raddle store is one directory per Sketch beside `board.json`, holding the approved proposal, the cold proposal and the Sketch's record; prune never removes it.
`lym board raddle <slug>` prints a stored proposal, found by the Sketch's slug or by any slug it filed.
A raddle run's finish never makes its entry a Sketch, chosen by the recipe's rows: a recipe with a `RaddleFile` row skips the end-as-Sketch finish a recipe with no Finalize row takes today.

### D11 — A beam's orch receives the Sketch and the proposal at its first start

When the pawl recipe's BeamUp row brings up a beam's orch and the raddle store holds a proposal that filed the beam's project, it mails that orch a hand-off naming the Sketch slug, `lym board get <project> --record` and `lym board raddle <project>`.
BeamUp records the hand-off as sent in its pawl run's state, so a re-invoked or later BeamUp sends none.
Rationale: at `lym beam create` the beam's orch is not yet up, and a notice to a worktree with no orch strand posts nothing.

### D12 — Every agent session a run spawns records its token reading

The entry's named sources, `lym report timing` and the token audits, hold no token cost in a hub, so recording it is part of this task.
`shuttleengine.SessionUsage` gains input, cache-write, cache-read and output tokens apart, which each engine fills from what it reads, an unreadable split staying unknown.
Shuttle reads the usage at every session end the provider can read, whatever the outcome, not only a done one.
One writer in shed's shared machinery records each session of a run, its forks included, as one JSON file per session id under the run's tracked `_lym/usage/`: row, role, model, the four token counts, start and end; a resumed session's later end overwrites its file.
Seed records the run's recipe and profile there, and Finalize records the landed change's lines added, lines deleted and files changed.

### D13 — `lym report cost` gives each recipe and profile's fixed and per-size cost

`lym report cost` reads the usage records of landed runs in its window and, per recipe and profile, reports the run count, the mean, and a least-squares fit of a fixed cost plus a cost per 100 changed lines, in dollars and in active hours, with the fit's correlation.
Dollars come from per-model prices in the report config key `prices`, and a cell under the report key `cost_min_runs` runs, default 5, reads "too few runs".
It has a JSON form, which the raddle agent reads.

### D14 — `lym report raddle` compares each proposal with what its tasks cost

`lym report raddle` lists each proposal in the raddle store with its predicted cost, the actual cost of the filed parts' landed runs, the parts not yet landed, and whether the approved proposal differs from the cold one.
Rationale: the operator's corrections and the cost error are what the raddle stencil is tuned from.

### D15 — The glossary marks `raddle` taken

`docs/research/weaving-glossary.md` gains a `raddle` row, which it lacks today, whose last cell reads `Taken:` with the recipe's one-line meaning, in the shape the `motif` row uses.

## Constraints

- `PATTERN-shed-recipe-registry`: the new engines are `Constructor` rows; the mode and model are setting lookups by key, never `Env` fields.
- `PATTERN-loom-contract`: the raddle verbs refuse on a recipe by its rows, never its name.
- `PATTERN-sole-parsers`: only `internal/raddleparser` (new) reads the proposal, and only `internal/discussionparser` reads and splices the decision record.
- `PATTERN-stencil-ownership`: the raddle stencil and proposal spec are read through `internal/stencilstore`.
- `PATTERN-shuttle-provider-seam`: token splits are read only under each engine's own package.
- `PATTERN-shed-shared-machinery`: the usage writer is one function in shed's packages.
- `PATTERN-durable-vs-ephemeral-state`: `_lym/usage/` and `_lym/raddle/` are named only in `internal/lymdirs`.
- `PATTERN-no-hard-coded-choice`: prices are config, never literals.
- `PATTERN-pair-parent`: filed entries take the Sketch's parent level through `hubgeom`'s one parent read.
- Fact: `orchcli.NotifyOrchAt` posts nothing to a worktree with no orch strand recorded (`internal/orchcli/notify.go`, this tree), which D11 rests on.
- Fact: no lym code records a run's token cost; only review rounds record a token reading (`internal/shedadapters/roundusage.go`, this tree), which D12 rests on.
- Fact: batten's Land prunes done entries for a run that lands no code, and removing an entry deletes its stored record (`internal/battencli/land.go`, `internal/boardengine/decision.go`, this tree), which D9 and D10 rest on.

## Auto-mode assumptions

Self-picks, reasons in the support log's question ledger: the agent runs the approval verb on the operator's word (D7); filed tasks are released (D9); every filed slug is new (D5); `opus[high]` (D4); part recipes `tapestry` and `darn` only (D5).

## Open risks

- A hub has no usage records until runs land under this binary, so early raddle runs see "too few runs" cells; the stencil has the agent say so in its cost lines rather than guess.
- D3 changes darn for discussed entries: a darn entry flagged discussed today would build from its stored record after this lands.
- A Codex session's rollout may not split cache-write from input; such a session's reading stays partly unknown and its cost is understated.
- Task `pawl-orch-landing` gives BeamUp a re-invoke policy; D11's sent-marker must hold under it.

## Acceptance criteria

- D1: a board task with recipe `raddle` runs through batten in its own worktree, its rows in order Record-Write, Raddle-Write, Raddle-File, and lands no code; the recipe loader accepts it under the loom contract's record-only form.
- D2: `lym board promote` with recipe `raddle` makes a Sketch a released task keeping its record, flag and `sketch` label in one write; a write making a Sketch a task with any other recipe, or dropping the label, is refused with the way forward; `IsSketch` reads such a task as no Sketch.
- D2: batten refuses to start a raddle run on an entry with no stored record, naming `motif`.
- D3: Record-Write for a discussed entry writes the stored record as the run's decision record, for darn, raddle and pawl alike; an undiscussed entry's record is rendered from its body as before.
- D4: Raddle-Write spawns its agent with the model of the loom key `raddle` and the raddle stencil.
- Review: D4 the raddle stencil names the commands the agent reads, states the tapestry fixed-cost rule, carries no cost figure, and points at the proposal spec.
- D5 and D6: a valid proposal passes; each check of D6 fails on a proposal that breaks it alone, naming the part or id; a part typed `sketch` and a part slug equal to the Sketch's are refused.
- D6: in auto mode a failing gate re-prompts the session and the third failure halts the run.
- D7: with the key off, `raddle-propose` stores the cold proposal and posts the waiting notice on its first pass only; `raddle-approve` before it, on an invalid proposal, or outside Raddle-Write is refused with its way forward; Raddle-File refuses a proposal edited after approval.
- D7: with the key on, Raddle-Write moves to Raddle-File once the gate passes, with no approval.
- D8: a changed `revised` stamp, a new task or project sharing a module label and absent from `board_checked`, or a taken slug fails the re-check; a weighed entry now done or gone passes, named; a failure at filing returns to Raddle-Write in a new session, and a third halts the run.
- D9: each shape files as stated with new slugs, the Sketch entry is done after the write, and `beam` creates the beam of the new project.
- D9: each part record passes the discussion check and holds exactly the decisions, criteria, risks and PATTERN changes its part carries, plus every `Added after Discussion` entry.
- D9: a Raddle-File interrupted after any step and run again finishes the remaining steps and repeats none; a `lym board upsert` creating a task from the raddle agent's session is still refused.
- D10: after the raddle run's Land prunes the Sketch, `lym board raddle` prints its proposal by the Sketch's slug and by a filed slug; a finished raddle run never makes its entry a Sketch.
- D11: BeamUp mails the hand-off to the orch of a beam whose project a stored proposal filed, once; a re-invoked BeamUp and a beam with no proposal send none.
- D12: a session of a run that ends done, stuck or halted writes one usage file keyed by its session id with the four token counts, and a resumed session's later end overwrites it; Seed writes the recipe and profile and Finalize the landed diff size.
- D13: `lym report cost` over fixture records gives the fitted fixed and per-100-line cost per recipe and profile, priced from the report key `prices`, and "too few runs" under the minimum.
- D14: `lym report raddle` over a fixture store reports predicted against actual cost, the unlanded parts, and whether the operator changed the cold proposal.
- Review: D15 the glossary's `raddle` row reads `Taken:` with the recipe's meaning.

## PATTERN changes

- `PATTERN-sole-parsers` — changes: Parsing plan, parts file, discussion, summary, recipe, halt note or raddle proposal, or classifying a plan ref: only its sole parser.
- `PATTERN-run-usage-record` — adds: Recording an agent session's tokens: only shed's one usage writer, one file per session under a run's `_lym/usage`; cost reports read only those.

## Notes for the plan writer

- `seedDiscussedRecord` in `internal/loomcli` already seeds a stored record for tapestry's Motif block; D3 can share it.
- `createSteps` in `internal/beamcli/create.go` already skips each step whose result exists, which D9's beam step relies on.
- `reportengine`'s timing computation already yields each run's active seconds for D13's hours.
