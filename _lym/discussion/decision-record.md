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
Neither new row carries `on_stuck`: a Stuck halts the run as `Plan-Write`'s does, and the operator's way back is a rerun or `lym loom goto --to Plan-Write`, which bypasses that row's entry gate and writes a solo plan.
The settled-verdict refusal of a goto onto a settled Plan-Review row names the plan writer this run ran, read from its history: `Parts-Plan-Write` when `Plan-Write` was recorded Skipped, `Plan-Write` otherwise.

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

Both rows take `plan_timeout_min` as their timeout.
`plan_lenses` stays `Plan-Write`'s alone: neither new producer forks lens subagents.

Rationale: the bench needs to vary the depth, the split size and the chair's model without code changes; Opus defaults keep the first bench a test of the split, not of a model change.

### D3 — The parts file

`Parts-Write` writes `_lym/parts/parts.md`, tracked like the plan; each part's cards go under `_lym/parts/<part>/` and the chair's join record to `_lym/parts/join.md` (D9).
The directory is named by the new sole parser `internal/partsparser` (new), which also parses the file; nothing else reads it.
The format is pinned in a new spec, `contracts/specs/loom-parts-spec.md` (new), whose LLM-facing subset is in the `Parts-Write` stencil (D10).

The file holds a title, a split note, one section per part and one shared-spots section:

- **Split note** — how the task splits, or why it does not (D4).
- **Per part:** a name; the decision ids it carries; what it takes in (from the record and from earlier parts); the behaviour it delivers; its exports, one backticked ref each, being the `plan:` handles and new file paths it creates and the existing members it re-signs or deletes that another part uses; the earlier parts it depends on; and its area, the packages and files its writer may read and change.
- **Shared spots** — each a file path no single part owns (a registry, a config template, `PATTERN.md`, a shared doc), with the parts that need it, what each needs from it, and its exports: the handles and new paths its cards will create that a part uses.

The `parts` gate (the `Parts-Write` row's `gates` entry, three attempts) runs the parser's check, which refuses:

- no part, or more parts than `parts_max`;
- a part name that is not a valid seat name under `seatengine.ValidateSeatName`, or a repeated name;
- a dependency on a part that is not earlier in the file, so file order is a topological order;
- an export claimed twice, by two parts, two shared spots, or one of each;
- a shared-spot path that a part's area lists as a file of its own; a shared spot inside a package an area names is allowed, and the part check (D7) keeps that part's cards off it;
- a decision id of the record that no part and no shared spot names, and a named id the record does not hold; the record's ids are read through a new `internal/discussionparser` export, never parsed by `internal/partsparser` (new).

Rationale: parsing the contracts lets Go build the seat table, feed each part check the names it may use, order the join and check delivered exports, so none of that rests on an agent's reading.

### D4 — Parts-Write

`Parts-Write` is a one-seat table on `shedadapters.NewMultiLLMProducerSourced`, role `parts`, segment plan, behind a commit decorator like `loomshed.NewPlanWrite` that commits `_lym/parts/` on Done.
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

On resume, a writer seat the probe did not find, which includes a done writer whose kept pane the probe removed, counts as finished when all its outputs exist and its Accept passes on them; the chair is told its part stands and the seat no longer answers.
A writer seat not found whose outputs do not all exist, or fail Accept, ends the step as today.
`Parts-Plan-Write`'s Accept for a writer seat runs the part check (D7).

`seatengine.Table` validation: a table either holds advisors only or writers only; a table holding writers has its lead's stencil include `seat-directive-writer-chair` (new) instead of `seat-directive-chair`, which keeps serving every advisor table unchanged.
The chair's hold deferral counts writer seats as it counts advisors.

Rationale: the engine already lets a seat write outputs, gates a seat on its own and lets a table builder decide the seats; what was missing is the done wake, a resume that keeps finished parts, the required default and directives that make the seat an author rather than an adviser.

### D6 — The Parts-Plan-Write table

`Parts-Plan-Write` is a seat table built at call time from the parts file, through a table source on `shedadapters.NewMultiLLMProducerSourced`:

- the lead is the chair, role prefix `parts-plan`, segment plan, its output `_lym/parts/join.md`, its inputs the record, the parts file and every part's `00-overview.md`;
- one writer seat per part in parts-file order, named after the part, its output the part's `00-overview.md` under `_lym/parts/<part>/`, its inputs the record, `relevant.md` and the parts file;
- every writer seat carries the `plan-part` gate (D7) from the row's `part_gates` key, with three attempts, and keeps its gate findings at a path of its own; the row's `gates` key holds the chair's `plan` gate, three attempts.

The fresh-spawn preparation runs `loomshed.NewPlanDirRotator` over the plan directory as `Plan-Write`'s does, its prior-plan amendment going to the chair, and moves every part directory and `join.md` into a new `_lym/parts/archive-<stamp>/`, keeping the parts file and earlier archives in place; it covers the card files the producer's own archive of seat output files would leave behind.
A post-Done commit decorator commits the pathspec `Plan-Write`'s commit uses, webster's durable directory and the plan amendment record included, plus `_lym/parts/`.
A Stuck from a failed writer or an exhausted gate halts the run (D1).
Bound: a resume keeps every finished part (D5); a writer whose strand is lost before it finished ends the step, and the rerun starts every part over.

Rationale: the existing sourced producer evaluates its table per call, so the seat count comes from data without a new producer type.

### D7 — A part's cards and the part check

A part writer writes a plan directory of its own in today's plan format at `_lym/parts/<part>/`: an `00-overview.md` with the same frontmatter rules, a Card Index and the part's cards numbered from 1.
`lym loom validate-plan --part <name>` is the writer's self-check, the `plan-part` gate's check set and the writer seat's Accept: the no-flag set over that directory, with these differences:

- the exports of the part's dependencies and of the shared spots the part is listed as needing count as declared for `handle-dangling` and `path-missing`;
- `caller-uncovered` and `pattern-changes-mismatch` are left to the joined plan's `plan` gate, since their verdicts rest on cards and overview sections outside the part;
- a card target that is a shared-spot path is a finding.

`--part` is a fifth mode, exclusive with the four flags, and refuses a name the parts file does not hold.

Rationale: the existing parser and check set read a part directory unchanged, so a part is caught before the join with the same rules the plan gate applies.

### D8 — The join

`lym loom join-parts` writes the joined plan into the plan directory through a new `internal/planparser` write path:

- it reruns each part's check (D7) and refuses with the findings if one fails, so a part edited after its gate is checked again;
- it refuses an export of a part that the part's cards do not create, re-sign or delete;
- it refuses a `plan:` handle or new file path that cards of two parts both create, exported or not;
- it refuses parts whose `language:` or `root:` differ;
- it numbers the parts' cards into one sequence in parts-file order from `first_card` 1, rewriting file names, headings, `Commit:` prefixes and card ids as `planparser.InsertCard`'s renumbering does;
- it writes `00-overview.md` with the shared frontmatter, a merged Card Index and each part's `## Shared Decisions` and `## Rename mechanic` carried over, and replaces any card files the plan directory held.

Each refusal names the part to fix and that its writer fixes it.
It refuses outside a run whose recipe holds a `Parts-Plan-Write` row, read from the recipe's rows, and when the parts file or a part's overview is missing.
Bound: it writes only the plan directory; a rerun discards whatever the chair added there after the last join.

Rationale: numbering and ordering across parts is mechanical, and the parts file's order already puts every exporter before its users, so Go does it exactly and the chair spends no turns renumbering.

### D9 — The chair's work and output

The chair writes no part of its own. In this order it:

- waits for every writer's done line, reading the parts file and each part's cards, not the code at large;
- checks each part's cards against its contract's delivered behaviour, and sends a mismatch back to that part's writer, which fixes its own part; a writer that no longer answers after a resume has its part fixed by the chair;
- answers a writer that raises a contract it finds wrong, and tells every other affected writer of the ruling;
- runs `lym loom join-parts` once every part stands, and never again after the next step begins;
- writes each shared spot as cards of its own with `lym loom insert-card`, creating the exports the parts file gives it, placed after the cards that make what the spot uses and before the first card that uses what it creates, reading only the code each spot needs;
- writes the overview's framing paragraph and `## PATTERN changes`, copied from the record;
- runs `lym loom validate-plan` until clean, then writes `_lym/parts/join.md` last: per part, whether its cards met the contract, every ruling it made, and where each shared-spot card went.

A seam found after the shared-spot cards are in is fixed in the joined plan by the chair, not by a rejoin.
The row's gate is the ordinary `plan` gate over the joined plan, so Plan-Review and Webster see an ordinary plan.

Rationale: the chair keeps judgement where Go cannot judge (behaviour against contract, shared spots), and its output is a record the bench can read.

### D10 — Parts-Write stencil

`loom-template-parts` (new) says:

- read the decision record and `relevant.md`, then only what its depth block allows;
- split by behaviour along module boundaries, each part one writer's coherent unit; a part too small to hold a card of its own, or two parts whose areas share most of their files, is too fine a split;
- write contracts, never cards: per part the fields of D3, and the shared spots no single part owns with their exports;
- order the parts so a part comes after every part it depends on, with no cycle;
- name every decision id in some part or shared spot;
- when it cannot split without reading code, write one whole-task part and say why;
- run the parts check before ending its turn.

It carries the parts format's LLM subset; the spec stays the Go contract.

### D11 — Part writer stencil

`loom-template-part-writer` (new), with `seat-directive-writer` and the shared plan-format block (D12), says:

- read the record, `relevant.md`, the parts file, and the code of its own area and of the shared spots it touches, never another part's code;
- write cards for its own part only, into its own directory, and never edit another part's cards or the plan directory;
- create every export its contract names, and use another part's or a shared spot's exports by the names the parts file gives;
- leave shared spots to the chair, naming in a card's intent what it needs from one;
- raise a contract it finds wrong to the chair and wait for the ruling instead of improvising;
- leave `## PATTERN changes` out of its part overview;
- write its `00-overview.md` last and run `lym loom validate-plan --part <name>` before ending its turn;
- after done, keep answering the chair; fix its own part when asked, run the part check again, and reply to the chair once it passes.

### D12 — Chair stencil and the shared plan-format block

`loom-template-parts-chair` (new), with `seat-directive-writer-chair` and the plan-format block, carries D9's steps as its instructions and says the chair reads cards, not code, beyond what the shared spots need.
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

## Constraints

- PATTERN-shed-recipe-registry: the two rows are `Constructor` rows in `internal/shedrecipe`, each value looked up by key, switched by `entry_gate`.
- PATTERN-sole-parsers: `internal/partsparser` (new) is the parts file's sole parser; the join is a new `internal/planparser` write path, the plan format's sole writer; decision ids come from `internal/discussionparser`.
- PATTERN-loom-contract: `lym loom join-parts` and `--part` refuse by the recipe's rows, never its name.
- PATTERN-stencil-ownership: every stencil is read at call time; the part writer and chair stencils point at the parts spec rather than restating it.
- PATTERN-durable-vs-ephemeral-state: `_lym/parts/` is tracked durable state.
- PATTERN-refusal-way-forward: every new refusal (parts check, part check, join verb) names its way forward, and a test reaches it.
- PATTERN-cwd-resolution: table builders take told paths; `internal/seatengine` stays geometry-blind.
- PATTERN-interactive-spawn and PATTERN-agent-messaging: writer seats are interactive tmux sessions talking to the chair through creel.
- No production table sets `Required` today; advisor tables keep today's notice, parent-notice and resume rules.
- `Plan-Write`'s behaviour, edges and gate are unchanged, and it stays the default.

## Auto-mode assumptions

- The model defaults, `parts_max`, the one-part fallback, no lens forks and committing the parts directory were picked without an operator; the support log's question ledger holds each pick and its reason.

## Open risks

- The board asks Discussion to settle the stencils' content (D10, D11, D12) with the operator before Plan; this run was autonomous, so that content is the writer's pick and the parent decides whether the operator approves it before Plan.
- A part writer working from contracts may still need to read a neighbouring part's code to get a seam right; the stencil forbids it, so the cost shows up as contract questions to the chair or as seam findings in Plan-Review.
- Shared-spot cards that a part needs before its own cards cannot be placed by `insert-card` after the join without the chair judging the position; a misplacement is caught by `uses-later-target` only where the card's refs show it.

## Acceptance criteria

- D1: with `plan_split` false a tapestry run records `Parts-Write` and `Parts-Plan-Write` Skipped and runs `Plan-Write`; with it true the run records `Plan-Write` Skipped after `Parts-Plan-Write`'s Done and reaches `Plan-Bouncer`; a goto onto `Plan-Write` in a split run runs it; the settled-verdict refusal names `Parts-Plan-Write` in a split run and `Plan-Write` otherwise.
- D2: the six keys load with their defaults, a `parts_depth` outside `light` and `medium` is refused, and each model key reaches its seat's model choice.
- D3: the parts check accepts a valid file, including a shared spot inside a package a part's area names, and refuses each listed case with a way forward; the `parts` gate re-prompts `Parts-Write` on a finding.
- D4: `Parts-Write` gets the block of its configured depth, moves a stale parts directory's entries into a new archive without nesting earlier ones before a fresh spawn, and commits the parts directory on Done with a clean tree.
- D5: a writer seat's accepted done queues one line to the chair; its failed end ends the step; its held turn end sends the parent nothing; a resume after a writer finished keeps the step going and tells the chair its part stands; a table mixing writers and advisors is refused; a writer table's lead must include `seat-directive-writer-chair`; advisor tables behave as before.
- D6: the table holds one writer seat per part in file order with the `plan-part` gate and the chair's inputs listing every part overview; a fresh spawn archives stale part directories and `join.md` but keeps the parts file; Done commits the plan's pathspec and the parts directory with a clean tree, a webster record moved by the rotation included.
- D7: `validate-plan --part` passes a part using a dependency's or a shared spot's export, refuses a dangling handle neither exports, refuses a card targeting a shared spot, and skips `caller-uncovered`.
- D8: `join-parts` numbers several parts into one sequence that passes the `plan` gate's format checks, refuses an unmet export, a handle or path two parts create, mismatched `language:`, a failing part and a missing part overview, and refuses on a recipe without the row.
- D9: the chair's row gate is the `plan` gate, and the step is Done only once `join.md` exists and the joined plan passes it.
- D10, D11 and D12: each new stencil and directive fills from its table's values with every marker set; the plan-format block names no step; `Plan-Write`'s composed prompt keeps its content after the block moves.
- D13: `lym report timing` puts both rows in the plan phase, both rows have the reinvoke interrupt policy, and the coverage guard pins both names.

## PATTERN changes

- `PATTERN-sole-parsers` — changes: Parsing plan, parts file, discussion, summary, recipe or halt note, or classifying a plan ref: only its sole parser. (test)

## Notes for the plan writer

- The plan-format block's markers must be filled in all three tables that include it; a part writer's `plan_dir` is its part directory.
