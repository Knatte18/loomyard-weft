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
A write that makes a note flagged `discussed` with a stored record a task must set recipe `raddle`, keep the stored record and the `discussed` flag, and carry the `sketch` label, which promotion restores when a note write dropped it;
any other such write is refused, naming `lym board promote <slug> --recipe raddle` as the way forward.
The board admits the `sketch` label on a task only while its recipe is `raddle`, and such a task is a raddle run's entry, not a Sketch, to `boardengine.IsSketch` and the README.
The promoted entry is released, so its parent's dobby starts the run.
Batten's start guard refuses a raddle run on an entry with no stored record, naming a `motif` run as the way forward.
It also refuses one on a Sketch whose slug the raddle store holds as another Sketch's, one with another recorded `created` stamp, or as a slug any stored proposal filed (D10), naming `lym board rename` as the way forward.
The board's refusal of a store slug (D10) meets such a Sketch at its promotion first, with the same way forward; this guard stays as the start's own check.
Bound: the rule keys on the `discussed` flag and the stored record, not on the `sketch` label, and admits exactly one recipe, so only a raddle run turns a Sketch into tasks, whether or not a note write dropped its label.
A discussed note that never became a Sketch, such as a discussed task made a note again, falls under the same rule by intent: its stored record would seed D3 as a Sketch's does.

### D3 — Record-Write seeds a discussed entry's stored record

For an entry flagged `discussed`, Record-Write copies the stored record to the run's decision record and writes the support log, instead of rendering the record from the entry's body.
This holds for every recipe with a Record-Write row: darn takes a discussed entry's record as final, and a pawl run takes a raddle-filed project's record, the Sketch, as its beam's project record.
Rationale: raddle files every part and project discussed (D9), and each must build from the record it was given.

### D4 — The raddle agent is a cold role of its own

Raddle-Write spawns one agent with stencil `contracts/stencils/raddle/raddle-template-agent.md` (new) and the loom config key `raddle`, default `opus[high]`, set apart from the loom key `discussion`.
It reads the run's decision record, `lym board list` and the entries touching the Sketch's packages or module labels, `lym loom peers`, `lym report cost` and `lym report timing`, and only the code needed to place each part: the packages and files the Sketch names, which packages each part changes, roughly how much, and who calls whom.
It designs nothing and asks the operator nothing until its proposal has passed the checks of D6.
The stencil points at the proposal spec and restates none of it; it carries the rule that a tapestry's large fixed start cost often makes fewer, larger parts or one tapestry cheaper, and no cost figure.
It also carries the rule that the agent's cost lines say when a cell they rest on reads "too few runs", rather than guess a figure for it.

### D5 — The proposal is one YAML file with a spec and a sole parser

The agent writes `_lym/raddle/proposal.yaml`, in the format `contracts/specs/raddle-proposal-spec.md` (new) pins, parsed only by `internal/raddleparser` (new), which imports the standard library and `internal/discussionparser` only.
It holds:

- `shape`: `tapestry`, `tasks`, `project` or `beam`, and for `project` and `beam` the project's slug and title.
- `board_checked`: each board entry the agent weighed, with its `revised` stamp.
- `parts`: per part, its slug, title, type label, recipe (`tapestry` or `darn`), profile, the Sketch decision ids it builds, the packages it changes with a rough size, its contract (takes in, delivers, exports), its `depends_on`, the Open risks and Acceptance criteria it carries, the PATTERN changes it carries, and its expected cost.
- `cost`: the proposal's expected cost, the next-best split described in one line with its expected cost, and the `lym report cost` cells both rest on.

Shape `tapestry` has exactly one part, which builds every decision.
Every filed slug, part or project, is new, free on the board and in the raddle store (D10), and differs from the Sketch's, so no filed entry shares a slug with the Sketch's motif or raddle runs or with an older stored proposal.

### D6 — Gate `raddle` checks the proposal

The checks:

- every Sketch decision is built by at least one part, every Acceptance criterion and Open risk is carried by at least one part, and every PATTERN change is carried by exactly one part;
- each part's carried criteria name every decision id it builds, so each part record passes the discussion check;
- every decision id a part's carried Acceptance criterion or Open risk names is built by that part or by a part on its `depends_on` path;
- `board_checked` holds every open task or project, other than the run's own entry, that carries a module label the Sketch carries;
- `depends_on` is acyclic and names only parts, entries on the board, or `board_checked` entries since gone from it, which D8 counts as landed;
- no two parts without a `depends_on` path between them list a common package;
- each recipe is `tapestry` or `darn`, each profile exists for its recipe, each type label is in the board's vocabulary and is not `sketch`, and each new slug is free on the board and in the raddle store;
- both cost lines are present.

In auto-approve mode the gate holds Raddle-Write's handoff, re-prompting the live session up to 3 attempts and then halting the run for its parent, like Discussion-Write's gate.
In interactive mode `lym loom raddle-propose` and `lym loom raddle-approve` run the same checks (D7).
Raddle-File runs them again before filing, in both modes (D9).

### D7 — Interactive approval by default, auto-approve by a key

The loom config key `raddle_auto_approve`, default `false`, chooses the mode, and a run profile's raddle block may set it.
Raddle-Write reads the key once, at its first spawn in a run, and records the mode in the run's state;
every later Raddle-Write session and Raddle-File read that recorded mode, never the key.

- Auto: the session is autonomous; it ends on the proposal file, the gate runs, and the row goes to Raddle-File with no approval.
  The run's first gate pass stores the proposal as the cold proposal, committed under `_lym/raddle/`, and the last gate-passing proposal is the approved one.
- Interactive: the session ends on the approval record.
  The agent runs `lym loom raddle-propose` once its proposal is complete; it runs D6's checks and returns their findings.
  Its first passing call in a run stores the proposal as the cold proposal, committed under `_lym/raddle/`, and its first passing call in each Raddle-Write session posts a notice to the parent orch that a proposal waits for the operator.
  The agent then puts the proposal to the operator, and revises the file while the operator questions or corrects the split, order, recipes or profiles.
  On the operator's explicit approval it runs `lym loom raddle-approve`, which runs D6's checks and D8's re-check and on success records the approval with the proposal's hash; the session's end takes the row to Raddle-File.

Bound: both verbs refuse outside a run whose current row is Raddle-Write, and `raddle-approve` refuses until the current Raddle-Write session has a passing `raddle-propose`, on an invalid proposal and on a moved board, each naming what to fix, so the waiting notice goes out in every session that approves, the first after a return included.
The verbs cannot tell the operator from the agent, so, as with `lym loom sampler-done`, an approval with no operator input is prevented only by the stencil; the auto key is the sanctioned way to file without one.
A key changed after Raddle-Write's first spawn, such as by a local-layer config write in the pair, changes no mode of the run.
In interactive mode Raddle-File files only a proposal whose hash matches the recorded approval.
The cold proposal is stored once per run and never overwritten.

### D8 — The proposal is checked against the board again at approval and at filing

The re-check passes only when:

- every `board_checked` entry still on the board has the same `revised` stamp; one now done, or gone from the board, counts as landed and is named to the agent without failing the check;
- no open task or project absent from `board_checked`, other than the run's own entry, carries a module label the Sketch carries, whenever it was created or promoted;
- every new slug is still free on the board and in the raddle store.

`lym loom raddle-approve` runs it, and Raddle-File runs it before its first step and again inside its board write (D9).
A failure in Raddle-File is Stuck back to Raddle-Write with the moved entries named, in a new session in the recorded mode (D7).
The return clears the recorded approval, so in interactive mode Raddle-File files only after a `raddle-approve` made after the return, whether or not the proposal changed.
Raddle-File allows 2 returns per run, counting its D6 and approval-hash returns (D9) with the re-check's, and then halts for the parent.
Raddle-File drops a `depends_on` on an entry gone from the board.
Bound: the board keeps no status for a removed entry, and its dependency rule already reads a slug it no longer holds as pruned done (`internal/boardengine/dependency.go`), so the re-check cannot tell a landed entry from one `lym board remove` deleted before it landed;
a part depending on such an entry is filed released with that dependency dropped.

### D9 — Raddle-File stages, files in one board write, creates the beam, then ends the run's entry

Raddle-File is a Go row with no agent.
It first checks whether step 3 is done: the raddle store holds a directory for the Sketch's slug whose recorded `created` stamp is the run's entry's, carrying the approved proposal's hash, which only step 3's own board write puts there (D10).
Until then, before step 1 it runs D6's checks on the proposal, in interactive mode matches the proposal's hash against an approval recorded after the latest return (D7, D8), and runs D8's re-check;
a failing D6 check, or a hash that matches no such approval, returns to Raddle-Write as a D8 failure does, naming the failing checks or the mismatch instead of moved entries, and counts among its returns.
Steps 1 and 2 write only into the run's `_lym/raddle/`, each output carrying the approved proposal's hash, and either step is skipped only when its outputs carry the hash of the proposal now approved, so outputs written for an earlier proposal are rewritten.
A D8 return deletes the step 1 and step 2 outputs of the failed attempt.
Every other step is skipped when its result is already there, so a retry after a crash finishes the steps it missed.

1. Stage the store: the approved proposal, the cold proposal, a copy of the Sketch's record and the run's entry's `created` stamp, for the raddle store (D10).
2. Part records: a record `decision-<slug>.md` for each filed entry, spliced from the run's decision record by a new `internal/discussionparser` function.
   A part's record holds its Goal and Scope from its contract and packages, the decisions it builds copied whole with their ids, every `### Added after Discussion` entry, the Sketch's Constraints and Auto-mode assumptions, and the Open risks, Acceptance criteria and PATTERN changes it carries.
   A project's record, and the `tapestry` shape's one part's record, is the Sketch's record whole.
3. Board write, one `boardengine` write under the board lock, after D8's re-check, through a new `boardengine` write that files entries with their decision records:
   - `tapestry` and `tasks`: each part is filed as a discussed task at the Sketch's own project level.
   - `project` and `beam`: the project is filed at that level with no `sketch` label, discussed, and each part as a discussed task in it.
   - Each filed entry's `decision-<slug>.md` is stored from its step 2 record and its `discussed` flag set in the write that creates the entry.
   - The staged store is written into the raddle store in the same write, with the approved proposal's hash and the filed slugs, so the write leaves its own done marker; the write's board sync commits the store with the filed entries (D10).
   - Filed tasks carry their recipe, profile, type label, the Sketch's module labels and `depends_on`, and a body naming the Sketch slug, the part's contract and packages.
     For `tapestry`, `tasks` and `project` they are released, since the approval or the auto key is the operator's release; for `beam` they are filed unreleased, so no dobby starts them before their project is a beam.
   - The run's entry stays open.
4. Beam: for `beam`, `lym beam create` for the new project, acting from the parent worktree.
5. Release: for `beam`, one board write, after step 4 made the project a beam, releases each filed part that is still on the board and unreleased, and skips a part that is absent from the board or done, since a released part can land and a Land prune it before a retry;
   the step's done marker is that no filed part still on the board is unreleased.
6. Notice: one notice to the parent orch naming what was filed, recorded as sent in the row's state.
7. End: one board write marks the run's entry done and clears its `notify_on_land`, which the run lock admits as a status change and a landing-notice change; step 6's notice tells the parent orch instead.
   The entry done is its done marker.

Bound: Raddle-File writes the board as the run's Go code with no strand name, so the board's refusal of a run agent creating a task still holds for the raddle agent's own `lym board` calls.
The run's entry stays open until step 7 because origin reads a done entry as an ended run, whose resume is refused, whose pair the hub reconcile tears down and whose later record commits a weft replay sets aside, since the replay spares only a run that entered Finalize or that the prime records as done or at Land, and a raddle run before step 7 is neither.
A halt before step 7 therefore shows on the run's entry like any run's halt and resumes.
Step 7 is the run's last write, so origin ends a run with nothing left to do; after it the run's later status writes skip the entry, its finish writes nothing to it, and batten's Land prunes it with its `decision-<slug>.md`, its `notify_on_land` cleared; the raddle store keeps the copy.

### D10 — The raddle store on Bolt outlives prune

The raddle store is one directory per Sketch beside `board.json`, named by the Sketch's slug and holding the approved proposal with its hash, the cold proposal, the Sketch's record, the Sketch's `created` stamp and the filed slugs; prune never removes it.
A store directory is a run's when its slug and recorded `created` stamp are the run's entry's.
The store stays keyed by slug, and no slug in it is reused: D2 refuses a raddle run on a Sketch whose slug the store holds as another Sketch's or as a filed slug, and D5, D6 and D8 require every filed slug free in the store as well as on the board.
The board's new-slug rule, `checkNewSlugs` in `internal/boardengine/slugrule.go`, which judges every write that creates a task or project, renames an entry or makes a note a task or project, also refuses a slug the store holds as a Sketch's slug or as a filed slug, whether or not the pair slug seam is set.
Its way forward names a slug that both the seam and the store admit;
for a note promotion, which keeps the note's slug, it names `lym board rename` of the note before the promotion, since a note's rename is not judged, and a filed part made a note again, such as by a motif run's end, takes the same way.
It reads the store as it stood before the write, so D9 step 3's write is not refused by the filed slugs it records, and it exempts the entry whose `created` stamp the store records for its own Sketch slug, so a live raddle run's Sketch is never refused.
`lym board raddle <slug>`, D11's lookup of the store by the beam project's slug and D14's listing rely on this refusal, so a slug in the store always names the entry that filed or was raddled under it.
Raddle-File writes it inside D9 step 3's board write.
That write, like every board write, commits nothing itself and spawns the detached board sync, whose commit stages every change in the Bolt, so the store directory reaches origin in the board sync's commit with the filed entries.
The board's Bolt replayer, `boardengine`'s `FabricReplay`, claims the store's paths and admits them to its design-file rule, both of which refuse a path holding a `/` today, and replays each store file by that rule: a file origin does not hold lands, so a concurrent board write from another machine, which writes no file of this Sketch's store, replays beside it.
The replay sets aside a store directory's files, reporting them with the entry, when it refuses or loses the add of any entry the directory records as filed, so origin never holds step 3's marker without the entries it marks.
`lym board raddle <slug>` prints a stored proposal, found by the Sketch's slug or by any slug it filed.
A raddle run's finish never makes its entry a Sketch, chosen by the recipe's rows: a recipe with a `RaddleFile` row skips the end-as-Sketch finish a recipe with no Finalize row takes today.

### D11 — A beam's orch receives the Sketch and the proposal at its first start

When the pawl recipe's BeamUp row brings up a beam's orch and the raddle store holds a proposal that filed the beam's project, it mails that orch a hand-off naming the Sketch slug, `lym board get <project> --record` and `lym board raddle <project>`.
BeamUp mails the hand-off, then records it as sent in its pawl run's state.
An interruption between the two lets a re-invoked BeamUp mail a second identical hand-off, which D11 accepts, and a BeamUp re-invoked or run later after the marker is recorded sends none.
Rationale: at `lym beam create` the beam's orch is not yet up, and a notice to a worktree with no orch strand posts nothing.

### D12 — Every agent session a run spawns records its token reading and cost

The entry's named sources, `lym report timing` and the token audits, hold no token cost in a hub, so recording it is part of this task.
`shuttleengine.SessionUsage` gains input, cache-write, cache-read and output tokens apart, which each engine fills from what it reads, an unreadable split staying unknown.
The Codex engine gains a `UsageReader` over the rollout's last `total_token_usage`: input is `input_tokens` less `cached_input_tokens`, cache-read is `cached_input_tokens`, output is `output_tokens`, which already includes `reasoning_output_tokens`, and cache-write is `cache_write_input_tokens` where the total carries it and zero otherwise, never unknown.
Shuttle reads the usage at every session end the provider can read, whatever the outcome, not only a done one.
One writer in shed's shared machinery records each session of a run, its forks included, as one JSON file per session id under the run's tracked `_lym/usage/`: row, role, model, the four token counts, the dollar cost, start and end; a resumed session's later end overwrites its file.
At the session's end the writer locates the session by its id and reads its messages, its forks' included, through the engine's `TranscriptReader` from `shuttleengines.RecordReaders`, and prices each message at its own model through the engine's `MessagePricer`; the cost is unknown for a model with no price.
For an engine with no `TranscriptReader`, such as Codex, it takes the four counts from `SessionUsage` and leaves the cost unknown.
A locate or read failure of any of its transcripts, a fork's included, such as one that is gone or unreadable, never fails or stops the session or the run:
the writer logs it at Warn and writes the session's file from the four counts `SessionUsage` holds, each unknown where it could not read them, with the cost unknown, as it does for an engine with no `TranscriptReader`, and a failed write of the file is logged and stops nothing.
`model` is the model of the session's own transcript, and a session whose messages, a fork's included, ran on more than one model also records each model's four token counts and cost.
The round usage record of a review round keeps its own per-half token reading, a summary of the round that no cost report reads; a cost report reads token counts and costs only from `_lym/usage/`.
Seed records the run's recipe and profile there and whether the run started from a stored record, and Finalize records the landed change's lines added, lines deleted and files changed.

### D13 — `lym report cost` gives each run cell's fixed and per-size cost

`lym report cost` reads the usage records of landed runs in its window and, per recipe, profile and whether the run started from a stored record, reports the run count, the mean, and a least-squares fit of a fixed cost plus a cost per 100 changed lines, in dollars and in active hours, with the fit's correlation.
A run with no Finalize size record, such as a motif or raddle run, enters its cell's mean only, so a cell of such runs reports its mean and no fit.
Dollars are the sum of a run's recorded session costs, and a run with a session of unknown cost enters the hours alone.
A cell under the report key `cost_min_runs` runs, default 5, reads "too few runs".
It has a JSON form, which the raddle agent reads.

### D14 — `lym report raddle` compares each proposal with what its tasks cost

`lym report raddle` lists each proposal in the raddle store with its predicted cost, the actual cost of the filed parts' landed runs, the parts not yet landed, and whether the approved proposal differs from the cold one.
Rationale: the operator's corrections and the cost error are what the raddle stencil is tuned from.

### D15 — The glossary marks `raddle` taken

`docs/research/weaving-glossary.md` gains a `raddle` row, which it lacks today, whose last cell reads `Taken:` with the recipe's one-line meaning, in the shape the `motif` row uses.

## Constraints

- `PATTERN-shed-recipe-registry`: the new engines are `Constructor` rows; the model is a setting lookup by key, and the mode one at Raddle-Write's first spawn, read from the run's state after it (D7), never `Env` fields.
- `PATTERN-loom-contract`: the raddle verbs refuse on a recipe by its rows, never its name.
- `PATTERN-sole-parsers`: only `internal/raddleparser` (new) reads the proposal, and only `internal/discussionparser` reads and splices the decision record.
- `PATTERN-stencil-ownership`: the raddle stencil and proposal spec are read through `internal/stencilstore`.
- `PATTERN-shuttle-provider-seam`: token splits are read, and messages priced, only under each engine's own package; a dollar cost comes only from the engine's `MessagePricer`, never a price table of lym's own.
- `PATTERN-shed-shared-machinery`: the usage writer is one function in shed's packages.
- `PATTERN-durable-vs-ephemeral-state`: `_lym/usage/` and `_lym/raddle/` are named only in `internal/lymdirs`.
- `PATTERN-pair-parent`: filed entries take the Sketch's parent level through `hubgeom`'s one parent read.
- Fact: `orchcli.NotifyOrchAt` posts nothing to a worktree with no orch strand recorded (`internal/orchcli/notify.go`, this tree), which D11 rests on.
- Fact: no lym code records a run's token cost; only review rounds record a token reading (`internal/shedadapters/roundusage.go`, this tree), which D12 rests on.
- Fact: a Codex rollout's `token_count` usage carries `input_tokens`, `cached_input_tokens` and `output_tokens`, and Codex's token reports also name `reasoning_output_tokens` (`internal/shuttleengine/codexengine/rollout_test.go`, `docs/research/codex-review-bench-2026-10-25/report.md`, this tree); some rollout totals also carry `cache_write_input_tokens`, zero in every one recorded (`docs/research/codex-astra-context-parity-bench-2026-10-27/data/rollout-evidence.txt`, this tree), and others carry no cache-write count; D12 rests on both.
- Fact: `output_tokens` already includes `reasoning_output_tokens`: in each rollout total of the context-parity bench run under Codex CLI 0.164.0, `total_tokens` equals `input_tokens` plus `output_tokens`, with `reasoning_output_tokens` beside them (`docs/research/codex-astra-context-parity-bench-2026-10-27/data/rollout-evidence.txt` and its `report.md`, this tree), which D12's output reading rests on.
- Fact: origin reads a done entry as an ended run (`FabricReplay.Runs` sets `Ended` from `isDone`, `internal/boardengine/replay.go`), and an ended run's resume is refused (`resumeRefusal`, `internal/battencli/resumereconcile.go`), its pair is torn down by the hub reconcile (`teardownEnded`, `internal/pairteardown/originseams.go`), and its later record commits are set aside by a weft replay, which spares only a run that entered Finalize or that the prime records as done or at Land (`withoutFinalizeEntered`, `finalizeEnteredRead`, `internal/fabricengine/replay.go`), and `applyDesignFile` refuses a path that is not a plain file name (`internal/boardengine/replay.go`), this tree, which D9's step 7 and Bound rest on.
- Fact: a board write saves and renders under the board lock and then spawns a detached board sync (`boardCriticalSection` in `internal/boardengine/board.go`, `spawnSync` in `internal/boardengine/spawn.go`), whose commit stages every change in the Bolt (`commitDirty` in `internal/boardengine/sync.go`, `Bolt.Commit` in `internal/fabricengine/bolt.go`), and `FabricReplay.Claims` claims only root files, `board.json`, overviews and `design-` and `decision-` files, refusing a path with a `/` (`internal/boardengine/replay.go`), this tree, which D10 rests on.
- Fact: Claude prices each message through `shuttleengine.MessagePricer`, which `tools/tokencount` already uses, and Codex implements neither `UsageReader` nor `MessagePricer` in this tree (`internal/shuttleengine/claudeengine/pricing.go`, `tools/tokencount/cost.go`, this tree), which D12 and D13 rest on.
- Fact: batten's Land prunes done entries for a run that lands no code, and removing an entry deletes its stored record (`internal/battencli/land.go`, `internal/boardengine/decision.go`, this tree), which D9 and D10 rest on.
- Fact: `Board.StoreDecisionRecord` refuses an absent entry and a done one (`internal/boardengine/decision.go`, this tree), so D9 step 3 stores the part records through a new write that creates their entries.
- Fact: the run lock compares neither `Status` nor `NotifyOnLand` (`scopeChanged` in `internal/boardengine/runlock.go`, this tree), which D9 step 7 rests on.
- Fact: `Board.SetRunStatus` and `Board.MakeSketch` skip a done entry, and `Store.Prune` keeps a done entry carrying `notify_on_land` (`internal/boardengine/board.go`, `internal/boardengine/sketch.go`, `internal/boardengine/store.go`, this tree), which D9's Bound rests on.
- Fact: `checkNewSlugs` judges every write that adds a task or project, makes a note a task or project, or renames an entry, through the pair slug seam `Config.ValidateSlug` alone, checking nothing when that seam is nil, and reads no raddle store (`internal/boardengine/slugrule.go`, `internal/boardengine/config.go`, this tree), which D10's added refusal rests on.
- Fact: the board entry `pawl-orch-landing` gives Beam-Up the re-invoke interrupt policy, since the row spawns no agent and is idempotent (its D11, stored with that entry on the board, which records the policy in `loomshed.InterruptPolicies`), which D11's order of mail and marker rests on.

## Auto-mode assumptions

Self-picks, reasons in the support log's question ledger: the agent runs the approval verb on the operator's word (D7); filed tasks are released (D9); every filed slug is new (D5); `opus[high]` (D4); part recipes `tapestry` and `darn` only (D5); the promote rule keys on the `discussed` flag (D2); a removed entry counts as landed (D8); the run's entry is ended in Raddle-File's last step (D9); a beam's parts are released after the beam exists (D9); the raddle store stays keyed by slug and refuses a reused one (D2, D10); the round usage record keeps its own token reading (D12).

## Open risks

- A hub has no usage records until runs land under this binary, so early raddle runs see "too few runs" cells; the discussed cells, which price every raddle-filed part, stay so until such parts land, whatever the hub's history.
- D3 changes darn for discussed entries: a darn entry flagged discussed today would build from its stored record after this lands.
- Codex implements no `MessagePricer`, so a Codex session's dollar cost stays unknown and its runs enter `lym report cost` in hours alone.

<!-- lym:carry-over Discussion-Review -->
- Discussion-Review, round 3: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/discussion/round-3-review.md; fixer report: _lym/reviews/discussion/round-3-fixer-report.md.
  - step-5-marker-lost-when-released-part-lands (class design, severity MEDIUM, open in rounds 3)
  - store-slug-reuse-by-non-raddle-creators (class design, severity MEDIUM, open in rounds 3)
<!-- /lym:carry-over Discussion-Review -->

## Acceptance criteria

- D1: a board task with recipe `raddle` runs through batten in its own worktree, its rows in order Record-Write, Raddle-Write, Raddle-File, and lands no code; the recipe loader accepts it under the loom contract's record-only form.
- D2: `lym board promote` with recipe `raddle` makes a Sketch a released task keeping its record, flag and `sketch` label in one write, and restores the label on a discussed note a note write stripped of it; a write making a discussed note with a stored record a task with any other recipe, or without the `sketch` label, is refused with the way forward, whether the note carries the label or not; `IsSketch` reads such a task as no Sketch.
- D2: batten refuses to start a raddle run on an entry with no stored record, naming `motif`, and on a Sketch whose slug the raddle store holds as another Sketch's, with another `created` stamp, or as a filed slug, naming `lym board rename`.
- D3: Record-Write for a discussed entry writes the stored record as the run's decision record, for darn, raddle and pawl alike; an undiscussed entry's record is rendered from its body as before.
- D4: Raddle-Write spawns its agent with the model of the loom key `raddle` and the raddle stencil.
- Review: D4 the raddle stencil names the commands the agent reads, states the tapestry fixed-cost rule and the rule that cost lines name a "too few runs" cell rather than guess, carries no cost figure, and points at the proposal spec.
- D5 and D6: a valid proposal passes; each check of D6 fails on a proposal that breaks it alone, naming the part or id, including a PATTERN change no part carries and one two parts carry, a part carrying a criterion or risk that names a decision neither it nor a part on its `depends_on` path builds, a `board_checked` missing an open entry sharing a module label, and a slug the raddle store holds; a part typed `sketch` and a part slug equal to the Sketch's are refused.
- D6: in auto mode a failing gate re-prompts the session and the third failure halts the run.
- D7: with the key off, `raddle-propose` stores the cold proposal on the run's first pass only and posts the waiting notice on the first pass of each Raddle-Write session; `raddle-approve` before a passing `raddle-propose` in the current Raddle-Write session, the first session after a return included, on an invalid proposal, or outside Raddle-Write is refused with its way forward; Raddle-File refuses a proposal edited after approval.
- D7: with the key on, Raddle-Write moves to Raddle-File once the gate passes, with no approval; the run's first gate pass stores the cold proposal and a later pass leaves it.
- D7: the mode recorded at Raddle-Write's first spawn holds for the run: a key changed after it changes the mode of no later Raddle-Write session and of Raddle-File.
- D8: a changed `revised` stamp, an open task or project absent from `board_checked` and sharing a module label, whether created after the agent's board read or a note promoted with its old dates, or a slug taken on the board or in the raddle store fails the re-check; a weighed entry now done or gone passes, named; a failure at filing returns to Raddle-Write in a new session and clears the approval, so in interactive mode an unchanged proposal is not filed until approved again; a third return, re-check, D6 or approval-hash, halts the run.
- D6 and D8: a `depends_on` on a `board_checked` entry pruned before Raddle-File passes D6's filing-time run and is dropped, named.
- D9: each shape files as stated with new slugs, each filed entry stored with its record and flagged discussed in the one write, `beam` creates the beam of the new project, and the run's entry is done with `notify_on_land` cleared only by step 7, after the notice.
- D9: no `beam`-shape part is released before its project is a beam.
- D9: each part record passes the discussion check and holds exactly the decisions, criteria, risks and PATTERN changes its part carries, plus every `Added after Discussion` entry.
- D9: a Raddle-File interrupted after any step and run again finishes the remaining steps and repeats none; one interrupted after step 3 finds step 3 done by the raddle store's hash, also after a filed part has landed and been pruned, and skips D8's re-check; one interrupted after step 5 and run again once a released part has landed and been pruned finds step 5 done, skips the pruned part and goes on to steps 6 and 7; a halt between steps 3 and 7 leaves the run's entry open, and the run resumes through batten; after a re-check failure at step 3 and a changed, re-approved proposal, Raddle-File rewrites the staged store and part records and files the new proposal; a proposal edited after approval returns to Raddle-Write naming the mismatch; a `lym board upsert` creating a task from the raddle agent's session is still refused.
- D10: after the raddle run's Land prunes the Sketch, `lym board raddle` prints its proposal by the Sketch's slug and by a filed slug, also on another machine once it pulls the board, and also when another machine wrote the board concurrently, so the store commit is replayed onto origin; a replay that refuses a filed entry's add sets aside the store's files with it; a finished raddle run never makes its entry a Sketch.
- D10: `lym board upsert` creating a task or project, `lym board rename` and a note promotion under a slug the raddle store holds are each refused with a slug named that the pair slug rule and the store both admit, the promotion naming `lym board rename` of the note first; after a filed project is pruned, a new project of the same slug is refused rather than matched to the old proposal; D9 step 3's write files its new slugs, and a live raddle run's own Sketch is not refused.
- D11: BeamUp mails the hand-off to the orch of a beam whose project a stored proposal filed, then records it as sent; a BeamUp re-invoked after an interruption between the mail and the marker mails a second identical hand-off; a BeamUp re-invoked or run again after the marker is recorded, and a beam with no proposal, send none.
- D12: a session of a run that ends done, stuck or halted writes one usage file keyed by its session id with the four token counts and its dollar cost, priced per message through the engine's pricer and unknown for an unpriced model, and a resumed session's later end overwrites it; a session with a fork on another model prices each message at its own model and records each model's counts and cost; Seed writes the recipe, the profile and whether the run started from a stored record, and Finalize the landed diff size; a session whose transcript cannot be read still writes its usage file from the counts `SessionUsage` holds, unknown where it could not read them, with an unknown cost, and its run goes on.
- D12: a Codex session's usage reads input as `input_tokens` less `cached_input_tokens`, cache-read as `cached_input_tokens`, output as `output_tokens` and cache-write as `cache_write_input_tokens`, or zero for a total without it, from the rollout's last `total_token_usage`.
- D12: a review round still writes its round usage record's token readings, and `lym report cost` over a run whose round usage records and `_lym/usage/` files disagree sums the `_lym/usage/` files.
- D13: `lym report cost` over fixture records gives the fitted fixed and per-100-line cost per recipe, profile and stored-record flag, with dollars summed from the records' session costs; a cell of runs with no size record gives its mean and no fit; a cell under the minimum reads "too few runs".
- D14: `lym report raddle` over a fixture store reports predicted against actual cost, the unlanded parts, and whether the operator changed the cold proposal.
- Review: D15 the glossary's `raddle` row reads `Taken:` with the recipe's meaning.

## PATTERN changes

- `PATTERN-sole-parsers` — changes: Parsing plan, parts file, discussion, summary, recipe, halt note or raddle proposal, or classifying a plan ref: only its sole parser.
- `PATTERN-run-usage-record` — adds: Recording an agent session's tokens and cost per session under a run's `_lym/usage`: only shed's one usage writer, one file per session; a cost report reads token counts and session costs only from those files, never from a review round's usage record.

## Notes for the plan writer

- `seedDiscussedRecord` in `internal/loomcli` already seeds a stored record for tapestry's Motif block; D3 can share it.
- `createSteps` in `internal/beamcli/create.go` already skips each step whose result exists, which D9's beam step relies on.
- `reportengine`'s timing computation already yields each run's active seconds for D13's hours.
- `messageCost` in `tools/tokencount` already prices one message through the engine's pricer, which D12's writer can follow.
