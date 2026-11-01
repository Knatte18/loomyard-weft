# Decision record — Parts-Write and Parts-Plan-Write

## Goal

Two new tapestry producers write the plan in two steps, as an opt-in alternative to `Plan-Write`, so a later research task can bench them against it.
`Parts-Write` splits the task coarsely into parts with contracts.
`Parts-Plan-Write` runs one fresh writing session per part and a chair that joins their cards into one ordinary plan.
`Plan-Write` stays as it is and stays the default.

## Scope

In: everything D1 to D13 decide.

Out:

- Any bench or comparison run, and any new starting profile; the bench task builds its own profile.
- Changes to `Plan-Write`'s behaviour, to Discussion-Write, to Plan-Review, to Webster, and to the darn, pawl and sampler recipes.
- A review step for the parts file.
- A heavy reading depth for `Parts-Write`.

## Decisions

### D1 — Rows and the switch

Tapestry gains two rows between `Drift-Baseline` and `Plan-Write`: `Drift-Baseline` → `Parts-Write` → `Parts-Plan-Write` → `Plan-Write` → `Plan-Bouncer`.
`Parts-Write` and `Parts-Plan-Write` carry `entry_gate: tapestry.plan_split`; `Plan-Write` carries `entry_gate: tapestry.plan_split != true`.
`Drift-Baseline`'s `on_done` becomes `Parts-Write`, `Parts-Write`'s is `Parts-Plan-Write`, `Parts-Plan-Write`'s is `Plan-Write`, and `Plan-Write` keeps `Plan-Bouncer`.
With the flag off both new rows are recorded Skipped and the run reaches `Plan-Write` as today; with it on, `Parts-Plan-Write`'s Done enters `Plan-Write`, whose gate skips it to `Plan-Bouncer`.
Neither new row carries `on_stuck`: a Stuck halts the run as `Plan-Write`'s does, and the operator's way back is a rerun of the halted row.
`lym loom goto` moves a halted run only back, so a run halted at `Parts-Write` or `Parts-Plan-Write` cannot be moved onto `Plan-Write`; to fall back to a solo plan the operator turns `plan_split` off and goes to `Drift-Baseline`, whose Done then skips both new rows and enters `Plan-Write`.
The settled-verdict refusal of a goto onto a settled Plan-Review row names the plan writer this run ran, read from `Plan-Write`'s latest history entry: `Parts-Plan-Write` when that entry is Skipped, `Plan-Write` otherwise, so a split run recovered by a goto onto `Plan-Write` names `Plan-Write`.

Bound: each row reads the flag when the run enters it, and a goto bypasses only its target's gate.
So a flag flipped mid-branch routes the rest of the run by the new value: flipped off after `Parts-Write`, the run skips `Parts-Plan-Write` and writes a solo plan; flipped off during `Parts-Plan-Write`, `Plan-Write` runs after it and its rotation archives the joined plan.
A goto onto `Parts-Write` or `Parts-Plan-Write` with the flag off likewise ends in `Plan-Write`.
The flag is meant to be set before a run starts, in its profile or `run_settings`.

Rationale: the entry gate is the switch `Drift-Baseline` already uses, a skip spends no bounce budget, and putting the new rows ahead of `Plan-Write` leaves every edge out of `Plan-Write` unchanged.

### D2 — Config keys

`internal/loomengine/tapestry.yaml` gains, all hub-tunable and settable in a profile's tapestry block or a task's `run_settings`:

- `plan_split: false` — turns the two rows on and `Plan-Write` off.
- `parts_depth: light` — `Parts-Write`'s reading depth, `light` or `medium` (D4); any other value is refused when config loads.
- `parts_max: 6` — the most parts a parts file may hold; the parts check (D3) refuses more.
- `parts: opus[medium]` — model of `Parts-Write`.
- `part_writer: opus[medium]` — model of every part writer.
- `parts_chair: opus[medium]` — model of the chair; a separate key so the bench can try a lighter model.
- `parts_chair_timeout_min` — the chair's timeout; unset by default, which means twice the run's effective `plan_timeout_min`; a set value not above the run's `plan_timeout_min` is refused when config loads.

`Parts-Write` and every part writer take `plan_timeout_min` as their timeout.
The chair takes `parts_chair_timeout_min`, since it starts first and spans every writer, the fix rounds, the join and the shared spots, so it must outlast any writer.
`plan_lenses` stays `Plan-Write`'s alone: neither new producer forks lens subagents.

Rationale: the bench needs to vary the depth, the split size and the chair's model without code changes; Opus defaults keep the first bench a test of the split, not of a model change.

### D3 — The parts file

`Parts-Write` writes `_lym/parts/parts.md`, tracked like the plan; each part's cards go under `_lym/parts/<part>/` and the chair's join record to `_lym/parts/join.md` (D9).
The directory is named by the new sole parser `internal/partsparser` (new), which also parses the file; nothing else reads it.
The format is pinned in a new spec, `contracts/specs/loom-parts-spec.md` (new), whose LLM-facing subset is in the `Parts-Write` stencil (D10).

The file holds a title, a split note, one section per part and one shared-spots section:

- **Split note** — how the task splits, or why it does not (D4).
- **Per part:** a name; the decision ids it carries; what it takes in (from the record and from earlier parts); the behaviour it delivers; its exports, one backticked ref each, being the `plan:` handles and new file paths it creates and the existing members it re-signs or deletes that another part uses; the earlier parts it depends on; and its area, the packages and files its writer may read and change.
- **Shared spots** — each a file path no single part owns (a registry, a config template, `PATTERN.md`, a shared doc), with the parts that need it, what each needs from it, its exports: the handles and new paths its cards will create that a part uses, and the exports of parts its cards use.

The `parts` gate (the `Parts-Write` row's `gates` entry, three attempts) runs the parser's check, which refuses:

- no part, or more parts than `parts_max`;
- a part name that is not a valid seat name under `seatengine.ValidateSeatName`, a repeated name, or a name beginning with the archive directory prefix `archive-`, whose directory the D4 and D6 preparations would skip as an earlier archive; the way forward is to rename the part;
- a dependency on a part that is not earlier in the file, so file order is a topological order;
- a shared spot that uses an export of a part coming after the first part that needs the spot, since no card position satisfies both; the way forward is to reorder the parts or move the dependency into a part; a spot that uses an export of that first part itself is allowed, its cards going between that part's cards;
- an export claimed twice, by two parts, two shared spots, or one of each;
- a shared-spot path that a part's area lists as a file of its own; a shared spot inside a package an area names is allowed, and the part check (D7) keeps that part's cards off it;
- a decision id of the record that no part and no shared spot names, and a named id the record does not hold; the record's ids are read through a new `internal/discussionparser` export, never parsed by `internal/partsparser` (new).

`lym loom validate-parts` (new) runs that same check over `_lym/parts/parts.md` and prints its findings, read-only; the gate and the verb call one function.
It refuses outside a run whose recipe holds a `Parts-Write` row, read from the recipe's rows, and its refusal's way forward says this run writes no parts file, so there is nothing to check, and that its plan is checked with `lym loom validate-plan`.

Rationale: parsing the contracts lets Go build the seat table, feed each part check the names it may use, order the join and check delivered exports, so none of that rests on an agent's reading.

### D4 — Parts-Write

`Parts-Write` is a one-seat table on `shedadapters.NewMultiLLMProducerSourced`, role `parts`, segment plan, behind a commit decorator like `loomshed.NewPlanWrite` that commits `_lym/parts/` whenever the producer reports a non-empty output pointer, on Done and on a gate-failed Stuck alike.
Its fresh-spawn preparation moves every entry of `_lym/parts/` except earlier archives into a new `_lym/parts/archive-<stamp>/`, so an earlier attempt's parts file, part cards and join record never read as this session's output, archives never nest, and the commit's pathspec holds both the moves and the archive.
Its prompt carries the PATTERN and friction directives as `Plan-Write`'s does, and the block of its reading depth:

- `light` — the decision record, `relevant.md`, and `lym quarry glyphs` over the packages the record names;
- `medium` — light, plus `lym quarry toc` of each package the task touches, which gives its package doc and each file's header, and `lym quarry expand` of the types the record names, which gives their signatures and members.

The depth names one of two stencil blocks, read at call time and filled into the prompt.
At either depth a `Parts-Write` that cannot split without reading code reads no further: it writes a parts file of one part covering the whole task and says why in the split note, and the run goes on with one part writer.
There is no review of the parts file; the whole-plan Plan-Review catches what a bad split causes.

Rationale: Discussion-Write already read the code and wrote down what matters; a one-part fallback keeps the run moving and leaves a countable trace for the bench.
The `quarry` verbs' output was checked against this worktree's `lym` build: `toc` returns package doc and file headers but no signatures, and `expand` returns a type's signature and members.

### D5 — Writing seats in the seat engine

`seatengine.Seat` gains a writer kind beside the advisor: a writing assistant that writes a product of its own the chair joins.
A writer seat:

- is required: every end its gate and Accept do not pass ends the step, as a required advisor's does today;
- when it ends done with its gate passed and accepted, queues one line to the chair naming the seat and its outputs, so a chair waiting on its parts is woken; a required advisor today queues no line;
- keeps its pane after done and keeps answering the chair, as an advisor does;
- holds a turn end quietly, as an advisor does, so a writer waiting on the chair sends the parent no held-turn notice; a required advisor today notifies the parent;
- has its stencil include `seat-directive-writer` (new), which takes the advisor directive's markers.

On a resume with a live chair, a writer seat the probe did not find, which includes a done writer whose kept pane the probe removed, counts as finished when all its outputs exist and its Accept passes on them; the chair is told the writer was stopped at the interruption, that any fix the chair had asked of it is lost, and that the seat no longer answers.
Such a writer has no terminal result, so the resume calls its Accept with a zero `shuttleengine.Result`; the D7 part check reads only files on disk, the part directory and the parts file, so the zero result decides nothing.
A writer seat not found whose outputs do not all exist, or fail Accept, ends the step as today.
A resume without a live chair is a fresh spawn (D6 Bound).
`Parts-Plan-Write`'s Accept for a writer seat runs the part check (D7).

`seatengine.Table` validation: a table either holds advisors only or writers only; a table holding writers has its lead's stencil include `seat-directive-writer-chair` (new) instead of `seat-directive-chair`, which keeps serving every advisor table unchanged.
The chair's hold deferral counts writer seats as it counts advisors, and a writer seat whose run ended done still counts as outstanding while its kept session reads mid-turn, read the same way the deferral reads a live advisor's session, so a chair waiting on a fix it asked of a done writer is deferred.
Today `advisorWork.outstanding` skips every run that ended (`internal/seatengine/work.go`), which would let each fix round reach the parent as a stall.

Rationale: the engine already lets a seat write outputs, gates a seat on its own and lets a table builder decide the seats; what was missing is the done wake, a resume that keeps finished parts, the required default, a hold deferral that counts a done writer's fix, and directives that make the seat an author rather than an adviser.

### D6 — The Parts-Plan-Write table

`Parts-Plan-Write` is a seat table built at call time from the parts file, through a table source on `shedadapters.NewMultiLLMProducerSourced`:

- the lead is the chair, role prefix `parts-plan`, segment plan, its output `_lym/parts/join.md`, its inputs the record, the parts file and every part's `00-overview.md`;
- one writer seat per part in parts-file order, named after the part, its output the part's `00-overview.md` under `_lym/parts/<part>/`, its inputs the record, `relevant.md` and the parts file;
- every writer seat carries the `plan-part` gate (D7) from the row's `part_gates` key, with three attempts; a resolved gate entry is a closure with no part argument, so the table source resolves the `part_gates` entry once per writer seat through a resolver that takes the seat's part name, and the seat's check runs `validate-plan --part <name>`'s set;
- each writer seat's gate findings path is the run's gate findings file for the key `Parts-Plan-Write-<part>`, set by the table source, so findings sit in run state and never in the part directory the join reads;
- the row's `gates` key holds the chair's `plan` gate, three attempts.

The fresh-spawn preparation runs `loomshed.NewPlanDirRotator` over the plan directory as `Plan-Write`'s does, its prior-plan amendment going to the chair, and moves every part directory and `join.md` into a new `_lym/parts/archive-<stamp>/`, keeping the parts file and earlier archives in place; it covers the card files the producer's own archive of seat output files would leave behind.
A commit decorator, committing whenever the producer reports a non-empty output pointer, on Done and on Stuck alike, commits the pathspec `Plan-Write`'s commit uses, webster's durable directory and the plan amendment record included, plus `_lym/parts/`.
`MultiLLMProducer` maps a result in which a writer seat ended the step, a `SeatResult` with `EndedStep` set, to `shedengine.Stuck` with a non-empty output pointer, the ended seat's first output path, so the commit decorator commits the part directories, when that writer ended done with a failed gate or an Accept error, or was not found at resume with outputs missing or failing Accept.
The Stuck reason names the seat, its part, and its gate findings path or its Accept error, or that its outputs were missing at resume.
Every other way a writer seat ends the step maps from that seat's own `SeatResult` (`EndedStep`, `Outcome`, `StartError`), which `MultiLLMProducer` reads before the chair's result, so the stopped chair's outcome never decides these cases:
- a writer that timed out maps to Stuck, its first output path the pointer, with a reason naming the seat, its part and the timeout;
- a writer that failed to start maps to an error wrapping `shuttleengine.ErrNotStarted` carrying the writer's `StartError`;
- a writer that died maps to an error wrapping `shuttleengine.ErrAgentDied` naming the writer seat, not the chair.

The two error cases keep the run's transient classification retrying them, and a required advisor's step end keeps today's mapping.
For a writer table, `join` marks a writer seat that ended the step `EndedStep` even when the chair's end arrives first or together with it, so that writer's mapping decides; today `join` lets a chair that has also ended decide the step (`internal/seatengine/engine.go` `join`), which stays so for advisor tables.
A chair timeout with no writer ended maps as `Plan-Write`'s chair timeout does today.
Today the engine's `join` stops the chair on a required seat's end (`internal/seatengine/engine.go` `join`), `mapOutcome` turns the stopped chair into an error wrapping `ErrAgentDied` and never reads `EndedStep` (`internal/shedadapters/multillm.go`), and the commit decorator commits only on a non-empty pointer (`internal/loomshed/planwrite.go` `Call`), so without this mapping a failed writer would fail the run uncommitted with a reason naming the chair.
So a writer whose gate, Accept, outputs or timeout failed, or an exhausted chair gate, leaves the run Stuck at `Parts-Plan-Write` (D1); a writer that died or never started returns an error.
Bound: a resume with a live chair keeps every finished part (D5); a writer whose strand is lost before it finished ends the step, and the rerun starts every part over.
When the chair is not live at a reinvoke or rerun, the probe stops every writer and the fresh path runs this preparation, which archives every part directory (`internal/seatengine/resume.go` `Resume` needs a live chair; `internal/shedadapters/multillm.go` `Call` runs `prepareFreshSpawn` on the fresh path), so a chair crash, timeout or machine restart writes every part again, finished or not.

Rationale: the existing sourced producer evaluates its table per call, so the seat count comes from data without a new producer type; the writer Stuck mapping is the one change to the producer, so a writer's own failure halts the run with its findings committed.

### D7 — A part's cards and the part check

A part writer writes a plan directory of its own in today's plan format at `_lym/parts/<part>/`: an `00-overview.md` with the same frontmatter rules, a Card Index and the part's cards numbered from 1.
`lym loom validate-plan --part <name>` is the writer's self-check, the `plan-part` gate's check set and the writer seat's Accept: the no-flag set over that directory, with these differences:

- the exports of the part's dependencies and of the shared spots the part is listed as needing count as declared for `handle-dangling` and `path-missing`;
- `caller-uncovered` and `pattern-changes-mismatch` are left to the joined plan's `plan` gate, since their verdicts rest on cards and overview sections outside the part;
- a card target that is a shared-spot path is a finding.

`--part` is a sixth mode, exclusive with the four flags, and refuses a name the parts file does not hold.

Rationale: the existing parser and check set read a part directory unchanged, so a part is caught before the join with the same rules the plan gate applies.

### D8 — The join

`lym loom join-parts` writes the joined plan into the plan directory through a new `internal/planparser` write path:

- it reruns each part's check (D7) and refuses with the findings if one fails, so a part edited after its gate is checked again;
- it refuses an export of a part that the part's cards do not create, re-sign or delete;
- it refuses a `plan:` handle or new file path that cards of two parts both create, exported or not;
- it refuses parts whose `language:` or `root:` differ;
- it numbers the parts' cards into one sequence in parts-file order from `first_card` 1, rewriting file names, headings, `Commit:` prefixes and card ids as `planparser.InsertCard`'s renumbering does;
- it writes `00-overview.md` with the shared frontmatter, a merged Card Index and each part's `## Shared Decisions` and `## Rename mechanic` carried over, and replaces any card files the plan directory held.

Each refusal names the part to fix and says the chair has its writer fix it, or fixes it itself when that writer no longer answers (D9).
It refuses outside a run whose recipe holds a `Parts-Plan-Write` row, read from the recipe's rows, and when the parts file or a part's overview is missing.
It refuses unless the run's status state is running and its current producer is that row, read from the run status's state and current producer and the recipe's rows, never a row name literal.
It refuses unless the caller is the chair's strand: the verb asks the creel daemon, through a new request, who the calling process is, which the daemon resolves as it resolves a sender, and compares that strand name against the chair of the row's seat table.
On Linux that identity comes from the calling process's pane; elsewhere it is the environment's, as creel's own is, so off Linux the chair-only bound holds only against callers that do not claim the chair's strand name.
So a halted run (blocked, failed, paused or awaiting) at that row, a run moved onto it by goto before its resume, a part writer, the operator's shell, orch, a tackler and an arbiter are all refused, each refusal naming its way forward:
- a halted run at the row, one moved onto it by goto included: `lym loom resume`, after which the row's chair runs the join; a live chair is resumed and joins itself, and otherwise the fresh spawn archives every part, rotates the plan directory and has the new chair join;
- a caller that is not the chair: the join is the chair's alone, and a running row's chair runs it; past `Parts-Plan-Write` the joined plan is edited in place, for example with `lym loom insert-card`.

Bound: it overwrites only the plan directory, and only from the chair of a running `Parts-Plan-Write` row, so a halted or later run, a part writer, a Plan-Burler-Fix fixer and Webster cannot reach it; within the row a rerun discards whatever the chair added after the last join, which D9 keeps the chair from doing.

Rationale: numbering and ordering across parts is mechanical, and the parts file's order already puts every exporter before its users, so Go does it exactly and the chair spends no turns renumbering.

### D9 — The chair's work and output

The chair writes no part of its own. In this order it:

- waits for every writer's done line, reading the parts file and each part's cards, not the code at large;
- checks each part's cards against its contract's delivered behaviour, and sends a mismatch back to that part's writer, which fixes its own part; a writer that no longer answers after a resume has its part fixed by the chair; a part whose fix request was outstanding at a resume is still open, and the chair rechecks it against its contract and fixes it itself before running `join-parts`;
- answers a writer that raises a contract it finds wrong, and tells every other affected writer of the ruling;
- runs `lym loom join-parts` once every part stands, and never again once it has written anything into the plan directory after the join;
- writes each shared spot as cards of its own with `lym loom insert-card`, creating the exports the parts file gives it, placed after the cards that make what the spot uses and before the first card that uses what it creates, reading only the code each spot needs;
- writes the overview's framing paragraph and `## PATTERN changes`, copied from the record;
- runs `lym loom validate-plan` until clean, then writes `_lym/parts/join.md` last: per part, whether its cards met the contract, every ruling it made, and where each shared-spot card went.

A seam found after the chair has written into the plan directory after the join is fixed in the joined plan by the chair, not by a rejoin.
The row's gate is the ordinary `plan` gate over the joined plan, so Plan-Review and Webster see an ordinary plan.

Rationale: the chair keeps judgement where Go cannot judge (behaviour against contract, shared spots), and its output is a record the bench can read.

### D10 — Parts-Write stencil

`loom-template-parts` (new) says:

- read the decision record and `relevant.md`, then only what its depth block allows;
- split by behaviour along module boundaries, each part one writer's coherent unit; a part too small to hold a card of its own, or two parts whose areas share most of their files, is too fine a split;
- write contracts, never cards: per part the fields of D3, and the shared spots no single part owns with their exports and the part exports each uses;
- order the parts so a part comes after every part it depends on, with no cycle;
- name every decision id in some part or shared spot;
- hold at most `parts_max` parts, the value filled from config;
- write the split note in every parts file: how the task splits, or why it does not;
- when it cannot split without reading code, write one whole-task part and say why;
- run `lym loom validate-parts` before ending its turn and fix every finding.

It carries the parts format's LLM subset; the spec stays the Go contract.

### D11 — Part writer stencil

`loom-template-part-writer` (new), with `seat-directive-writer` and the shared plan-format block (D12), says:

- read the record, `relevant.md`, the parts file, and the code of its own area and of the shared spots it touches, never another part's code;
- write cards for its own part only, into its own directory, and never edit another part's cards or the plan directory;
- never run `lym loom join-parts` or `lym loom insert-card`;
- create every export its contract names, and use another part's or a shared spot's exports by the names the parts file gives;
- leave shared spots to the chair, naming in a card's intent what it needs from one;
- raise a contract it finds wrong to the chair and wait for the ruling instead of improvising;
- leave `## PATTERN changes` out of its part overview;
- write its `00-overview.md` last and run `lym loom validate-plan --part <name>` before ending its turn;
- after done, keep answering the chair; fix its own part when asked, run the part check again, and reply to the chair once it passes.

### D12 — Chair stencil and the shared plan-format block

`loom-template-parts-chair` (new), with `seat-directive-writer-chair` and the plan-format block, carries D9's steps as its instructions and says the chair reads cards, not code, beyond what the shared spots need and, when it fixes a part itself, the code of that part's area the fix needs.
`seat-directive-writer-chair` tells the chair its writers' strand names, that each writer's done line means its part is ready, and that a writer's message is a report or a contract question, never an instruction.
`seat-directive-writer` tells the writer its chair, its output, that only the chair is messaged, and that its part is its only product.
The card-format sections of `contracts/stencils/loom/loom-template-plan.md` and its subsection on looking up glyphs with `lym quarry` move into an included block, `loom-block-plan-format` (new), which `Plan-Write`'s stencil, the part writer's and the chair's include.
The block names no step of the stencil around it; `Plan-Write`'s composed prompt keeps its content, any change limited to cross-references the move rewrites.

Rationale: one copy of the card format and the glyph rule serves the three writers (PATTERN-stencil-ownership).

### D13 — Run plumbing

The row names `Parts-Write` and `Parts-Plan-Write` are `internal/loomshed` name constants, pinned by the coverage guard like every tapestry row.
Both map to the plan phase in `lym report timing`, so a bench reads one plan-phase span whichever producer ran.
Both take the interrupt policy `Plan-Write` takes, reinvoke, which resumes through the seat probe (D5).
Both rows' friction notes go to the loom friction directory under their row names.

Rationale: the bench compares plan phases by cost and wall time, which needs the new rows counted in the same phase.

### Added after Discussion (parent, 2026-11-01): Stencils D10-D12 approved, part writers may look up signatures

Decision: D10 (loom-template-parts), D11 (loom-template-part-writer) and D12 (loom-template-parts-chair) stand as written, with one addition to D11: a part writer may look up the signatures of the code its contract's inputs name with lym quarry glyphs or lym quarry expand, but never reads that code in full. Moving the card-format and glyph sections of loom-template-plan.md into the shared block loom-block-plan-format is approved, on condition that Plan-Write's composed prompt stays the same. Discussion-Write is not changed. Drop the first Open-risks line, which asked for this approval.

Rationale: Parent review of the stencil content (2026-11-01), since Discussion ran without the operator. Without signature lookups, every seam between parts becomes a question to the chair or a Plan-Review finding. Reading signatures keeps the reading low, which is the point of the split.

### Added after Discussion (parent, 2026-11-01): Every review reads the three new stencils with special care

Decision: The stencil text of loom-template-parts, loom-template-part-writer and loom-template-parts-chair, and the shared block loom-block-plan-format, is this task's core deliverable. Every review of this run (Discussion review, Plan review, Code review) reads that text line by line and checks it as a prompt an agent will follow: each instruction is unambiguous and can be done with the tools and files the agent has, nothing contradicts another stencil or the plan format, each stopping condition and hand-off between Parts-Write, part writer and chair is stated, and nothing invites the extra code reading the split is meant to avoid. The Plan cards and the code must carry the stencil text in full, never as a summary left to Webster.

Rationale: Operator, 2026-11-01: the stencils of the two producers must be reviewed very carefully. Discussion ran without the operator, so the reviews are the only close reading the stencils get.

### Added after Discussion (parent, 2026-11-01): Chair may read one part's code when it fixes that part; join-parts identifies its caller by strand name

Decision: D12's exception stands as written: when a part's writer no longer answers (D9, after a resume), the chair fixes that part itself and may read the code of that part's area only, and only for that fix. D8's chair-only check reads the caller's strand name the way lym already identifies a run session (LYM_STRAND_NAME, as the board's run-session refusal does) and compares it with the Parts-Plan-Write table's chair seat; no new creel daemon request, no pane resolution, and no change to internal/creelengine. Drop the off-Linux caller-identity limit sentence. Everything else in D8 stands.

Rationale: Parent, 2026-11-01: ending the row Stuck and rewriting a whole part after every reinvoke costs more than the chair reading one part's area once; the reading limit still holds in the normal path. The strand name already identifies a session everywhere in lym, so the bound holds on every OS with no new mechanism.

### Added after Discussion (parent, 2026-11-01): Plan-Write's prompt stays line for line; parts file is never edited by the chair; part check flags a missing export

Decision: (1) Plan-Write's composed prompt stays the same line for line, in today's order: card 12 must not move the glyph subsection or the two kept sentences. Split the shared text into as many included blocks as the order needs (such as a second block for the glyph subsection, included where it stands today), and the part writer and chair include the same blocks. (2) The chair never edits the parts file. ValidatePart and planparser.JoinParts compare each contract export with the part's created, re-signed and deleted refs by the canonical spelling the resolve pass gives, so a respelled handle still matches; drop the respell clause and the stencil bullets that let the chair respell, as finding F3 lists. (3) part-export-missing stays in ValidatePart as a fourth D7 difference: a part writer whose part lacks an export its contract names learns it from its own check and fixes its own cards before the join.

Rationale: Parent, 2026-11-01: Plan-Write is the baseline the bench measures the split against, so its prompt must not change at all. The parts file is Parts-Write's output and a canonical comparison makes respelling needless. Catching a missing export at the writer is cheaper than the chair finding it at the join and sending the part back.

### Added after Discussion (parent, 2026-11-01): The part check leaves pattern-line-unprescribed to the joined plan

Decision: D7's second difference extends to pattern-line-unprescribed: the part check runs without a decision record path, so neither pattern-changes-mismatch nor pattern-line-unprescribed runs on a part; both run on the joined plan, where the chair writes ## PATTERN changes. Card 7's Intent cites this entry as the licence for clearing DecisionRecordPath.

Rationale: Parent, 2026-11-01: PATTERN changes are the chair's, written after the join, so a part cannot meet a pattern check; checking them once on the joined plan loses nothing.

## Constraints

- PATTERN-shed-recipe-registry: the two rows are `Constructor` rows in `internal/shedrecipe`, each value looked up by key, switched by `entry_gate`.
- PATTERN-sole-parsers: `internal/partsparser` (new) is the parts file's sole parser; the join is a new `internal/planparser` write path, the plan format's sole writer; decision ids come from `internal/discussionparser`.
- PATTERN-loom-contract: `lym loom join-parts`, `lym loom validate-parts` and `--part` refuse by the recipe's rows, never its name.
- PATTERN-stencil-ownership: every stencil is read at call time; the part writer and chair stencils point at the parts spec rather than restating it.
- PATTERN-durable-vs-ephemeral-state: `_lym/parts/` is tracked durable state.
- PATTERN-refusal-way-forward: every new refusal (parts check, `validate-parts`, part check, join verb) names its way forward, and a test reaches it.
- PATTERN-cwd-resolution: table builders take told paths; `internal/seatengine` stays geometry-blind.
- PATTERN-interactive-spawn and PATTERN-agent-messaging: writer seats are interactive tmux sessions talking to the chair through creel.
- No production table sets `Required` today; advisor tables keep today's notice, parent-notice, resume and outcome-mapping rules, the D6 Stuck mapping covering writer seats only.
- `Plan-Write`'s behaviour, edges and gate are unchanged, and it stays the default.

## Auto-mode assumptions

- The model defaults, `parts_max`, the one-part fallback, no lens forks and committing the parts directory were picked without an operator; the support log's question ledger holds each pick and its reason.

## Open risks

- A part writer working from contracts may still need to read a neighbouring part's code to get a seam right; the stencil forbids it beyond signature lookups with `lym quarry glyphs` or `expand`, so the cost shows up as contract questions to the chair or as seam findings in Plan-Review.
- The parts check guarantees each shared spot a valid range in part order (D3), but the chair still judges the card position within it, inside one part included; a misplacement is caught by `uses-later-target` only where the card's refs show it.

<!-- lym:carry-over Discussion-Review -->
- Discussion-Review, round 2: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/discussion/round-2-review.md; fixer report: _lym/reviews/discussion/round-2-fixer-report.md.
  - chair-timeout-equals-writer-timeout (class design, severity MEDIUM, open in rounds 2)
  - failed-writer-ends-as-error-not-stuck (class design, severity MEDIUM, open in rounds 1, 2)
  - join-parts-unbounded-by-current-row (class design, severity BLOCKING, open in rounds 1, 2)
  - parts-write-self-check-has-no-command (class design, severity MEDIUM, open in rounds 2)
  - resume-mid-fix-loses-fix (class design, severity MEDIUM, open in rounds 2)
<!-- /lym:carry-over Discussion-Review -->

<!-- lym:carry-over Plan-Review -->
- Plan-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/plan/round-2-review.md; fixer report: _lym/reviews/plan/round-2-fixer-report.md.
  - card7-spec-extends-should-be-edit (class consistency, severity MEDIUM, open in rounds 2)
<!-- /lym:carry-over Plan-Review -->

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - seatengine-lost-writer-end-step-untested (class scope, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria

- D1: with `plan_split` false a tapestry run records `Parts-Write` and `Parts-Plan-Write` Skipped and runs `Plan-Write`; with it true the run records `Plan-Write` Skipped after `Parts-Plan-Write`'s Done and reaches `Plan-Bouncer`; a goto onto `Plan-Write` in a split run runs it; the settled-verdict refusal names `Parts-Plan-Write` in a split run, `Plan-Write` in a split run recovered by a goto onto `Plan-Write`, and `Plan-Write` otherwise.
- D2: the new keys load with their defaults, an unset `parts_chair_timeout_min` gives the chair twice the run's `plan_timeout_min` and a set one not above it is refused, `plan_timeout_min` reaches each writer, a `parts_depth` outside `light` and `medium` is refused, and each model key reaches its seat's model choice.
- D3: the parts check accepts a valid file, including a shared spot inside a package a part's area names, and refuses each listed case with a way forward; the `parts` gate re-prompts `Parts-Write` on a finding; `lym loom validate-parts` prints the same findings as the gate for the same file and refuses on a recipe without a `Parts-Write` row.
- D4: `Parts-Write` gets the block of its configured depth, moves a stale parts directory's entries into a new archive without nesting earlier ones before a fresh spawn, and commits the parts directory on Done with a clean tree; an exhausted parts gate leaves the run Stuck at `Parts-Write` with the parts directory committed and a clean tree.
- D5: a writer seat's accepted done queues one line to the chair; its failed end ends the step; its held turn end sends the parent nothing; the chair's held turn sends the parent nothing while a done writer is mid-turn on a fix; a resume with a live chair after a writer finished keeps the step going and tells the chair the writer was stopped and any fix asked of it is lost, a writer stopped mid-fix included; a table mixing writers and advisors is refused; a writer table's lead must include `seat-directive-writer-chair`; advisor tables behave as before.
- D6: the table holds one writer seat per part in file order with the `plan-part` gate and the chair's inputs listing every part overview; a fresh spawn archives stale part directories and `join.md` but keeps the parts file; Done commits the plan's pathspec and the parts directory with a clean tree, a webster record moved by the rotation included; a writer's exhausted gate or failed Accept leaves the run Stuck at `Parts-Plan-Write` with a clean tree and a reason naming the seat; a writer that timed out leaves the run Stuck with a reason naming the seat, its part and the timeout; a writer that failed to start returns an error wrapping `shuttleengine.ErrNotStarted` carrying its start error; a writer that died returns an error wrapping `shuttleengine.ErrAgentDied` naming the writer seat; a chair timeout after a writer ended the step is decided by the writer's mapping.
- D7: `validate-plan --part` passes a part using a dependency's or a shared spot's export, refuses a dangling handle neither exports, refuses a card targeting a shared spot, and skips `caller-uncovered`.
- D8: `join-parts` numbers several parts that use no shared-spot export into one sequence that passes the `plan` gate's format checks, refuses an unmet export, a handle or path two parts create, mismatched `language:`, a failing part and a missing part overview, refuses on a recipe without the row, refuses while another row is current, refuses a blocked run at the row, and refuses a caller that is not the chair.
- D9: the chair's row gate is the `plan` gate, and the step is Done only once `join.md` exists and the joined plan passes it.
- D10, D11 and D12: each new stencil and directive fills from its table's values with every marker set, `Parts-Write`'s `parts_max` included; the plan-format block names no step; `Plan-Write`'s composed prompt keeps its content after the block moves.
- D13: `lym report timing` puts both rows in the plan phase, both rows have the reinvoke interrupt policy, and the coverage guard pins both names.

## PATTERN changes

- `PATTERN-sole-parsers` — changes: Parsing plan, parts file, discussion, summary, recipe or halt note, or classifying a plan ref: only its sole parser. (test)

## Notes for the plan writer

- The plan-format block's markers must be filled in all three tables that include it; a part writer's `plan_dir` is its part directory.
