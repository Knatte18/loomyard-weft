# Card 14 — parts-plan-write-table

**Create:**
- `contracts/stencils/loom/loom-template-part-writer.md`
- `contracts/stencils/loom/loom-template-parts-chair.md`
- `plan:contracts/stencils#LoomTemplatePartWriter` -> `var LoomTemplatePartWriter []byte`
- `plan:contracts/stencils#LoomTemplatePartsChair` -> `var LoomTemplatePartsChair []byte`
- `plan:internal/loomengine#PartsPlanRole` -> `const PartsPlanRole = "parts-plan"`
- `plan:internal/loomengine#PartsPlanTable` -> `func PartsPlanTable(layout *lymcwd.Location, stencilsDir, specsDir string, cfg Config, reg modelspec.Registry, gateTiers string, runTimeout time.Duration, gates func(part string) (shuttleengine.GateSpec, error), accept func(part string) func(shuttleengine.Result) error, findingsPath func(part string) string) (seatengine.Table, error)`
- `plan:internal/loomengine#TestPartsPlanTable` -> `func TestPartsPlanTable(t *testing.T)`

**Extends:**
- `contracts/stencils#entries`

**Edit:**
- `contracts/stencils#TestStencils_TopLevelMarkers`

**Uses:**
- `plan:internal/partsparser#Parse`
- `plan:internal/partsparser#File`
- `plan:internal/partsparser#Dir`
- `plan:internal/partsparser#PartDir`
- `plan:internal/partsparser#PartOverview`
- `plan:internal/partsparser#JoinRecord`
- `plan:internal/loomengine#Config.PartsChairTimeout`
- `plan:internal/loomengine#TestPartsTable`
- `internal/loomengine#PlanTable`

**Intent:** Build the `Parts-Plan-Write` row's seat table from the parts file at call time (D6), and ship the part writer's and the chair's stencils (D11, D12) in full.

**Table.** `PartsPlanTable` reads and parses `partsparser.File(anchor)`; an absent file fails the table with the way forward `lym loom goto --to Parts-Write`, and a malformed one with the way forward `lym loom validate-parts`.
Both ways forward add that a resumed row whose chair or writers are still live keeps their strands, since the table fails before the probe can stop them, so the parent first removes each `parts-plan` strand of the task with `lym reed remove --name <strand>`.
`runTimeout` is shuttle's `run_timeout_min` as a duration, told by the caller so a 0 `plan_timeout_min` resolves to the effective plan timeout.
It returns a table with role prefix `PartsPlanRole`, segment plan, `Timeout` `cfg.PlanTimeout(runTimeout)`, which every writer takes, and:
- the lead, the chair: stencil `loom-template-parts-chair`, model `cfg.PartsChair` with `ModelKey{Module: "tapestry", Key: "parts_chair"}`, seat `Timeout` `cfg.PartsChairTimeout(runTimeout)`, whose error fails the table, output `partsparser.JoinRecord`, inputs the decision record, the parts file and every part's `PartOverview` in file order, and seat values `plan_dir` (`planparser.PlanDir`), `overview_path` (`planparser.PlanOverview`) and `join_path`;
- one writer seat per part in file order: `Name` the part, `Writer` true, stencil `loom-template-part-writer`, model `cfg.PartWriter` with `ModelKey{Module: "tapestry", Key: "part_writer"}`, output the part's `PartOverview`, inputs the decision record, `relevant.md` and the parts file, `Gate` from `gates(part)`, `Accept` from `accept(part)`, `GateFindingsPath` from `findingsPath(part)`, and seat values `part_name`, `plan_dir` (`partsparser.PartDir`) and `overview_path` (`partsparser.PartOverview`).
Table values are `decision_record_path`, `relevant_path`, `parts_path`, `parts_dir`, `specs_dir`, `gate_tiers` and `pattern_directive`, the last optional, plus `friction_directive` set empty and optional: each seat sets its own friction directive as a seat value when it renders, the chair's for the note `Parts-Plan-Write` and each writer's for `Parts-Plan-Write-<part>` in the loom friction directory, so the notes never collide; a render failure is logged and leaves the table's empty value.
A `gates(part)` error fails the table; it is a fail-loud wiring guard in `refusal-spec.md`'s sense, taking no reaching test, since the entry validates the `part_gates` list once at construction and its per-part closure has no production error path.
Skills are `planSkills`; no seat forks.
The table carries no chair gate; the producer entry sets it.

`TestPartsPlanTable` (tier0, built as `TestPartsTable` is, over a written parts file of two parts) covers: one writer seat per part in file order with its name, output, inputs, gate and findings path, and an Accept that is the told one, observed by a told `accept` hook that records the part it was built for and returns an Accept recording its own call, which the test calls on each seat; the chair's inputs listing every part overview and its output the join record; the writers' table timeout and the chair's seat timeout, unset (twice `plan_timeout_min`, and twice `runTimeout` with a 0 `plan_timeout_min`) and set, and a set one not above `runTimeout` with a 0 `plan_timeout_min` failing the table; `part_writer` and `parts_chair` reaching their seats' model choices; each seat's composed prompt from `seatengine.Prompts` holding its part name, its plan directory and the plan-format block, with no `{{`; and an absent and a malformed parts file refused with their ways forward.
`TestStencils_TopLevelMarkers` gains `loom-template-part-writer` with `decision_record_path`, `edit_directive`, `friction_directive`, `overview_path`, `parent_directive`, `part_name`, `parts_path`, `pattern_directive`, `plan_dir`, `relevant_path` and `specs_dir`, and `loom-template-parts-chair` with `decision_record_path`, `edit_directive`, `friction_directive`, `join_path`, `overview_path`, `parent_directive`, `parts_dir`, `parts_path`, `pattern_directive`, `plan_dir` and `specs_dir`; the two stencils embed beside `LoomTemplatePlan`, with entries in `entries`.

`contracts/stencils/loom/loom-template-part-writer.md`, in full:

````markdown
<!-- This is a part writer's opening prompt: one writer seat per part of the parts file, in the Parts-Plan-Write row's table that internal/loomengine's PartsPlanTable builds, filled by internal/seatengine from the seat's value map.
     part_name, plan_dir (the part's own directory) and overview_path (its 00-overview.md) are seat values; decision_record_path, relevant_path, parts_path and specs_dir are table values;
     parent_directive and edit_directive are rendered by the seat engine, and pattern_directive and friction_directive are optional.
     It includes seat-directive-writer and loom-block-plan-format, each rendered from the same map; the block's gate_tiers and specs_dir come from the table. -->

# Part writer — turn one part's contract into plan cards

You are a part writer: one of several sessions that each turn one part of the task into plan-format cards.
Your part is `{{.part_name}}`, and its contract is the `## Part: {{.part_name}}` section of the parts file.
A chair joins every part's cards into one plan, writes the cards of the shared spots, and rules on contracts.

{{.parent_directive}}
{{.edit_directive}}
{{template "seat-directive-writer"}}
{{.pattern_directive}}
{{.friction_directive}}
## Step 1 — Read your contract

Read `{{.decision_record_path}}`, the decision record: the design and scope of the whole task.
Never read the support log or the board.
Read `{{.relevant_path}}` if it exists: a head start on the code that never adds scope.
Read `{{.parts_path}}`, the parts file, whose format `{{.specs_dir}}/loom/loom-parts-spec.md` defines.
Your contract is your part's section: the decisions it carries, what it takes in, the behaviour it delivers, its exports, the parts it depends on and its area.
The shared spots whose `Needed by` names your part are the ones it touches.

## Step 2 — Read your area

- Read the code of your area and of the shared spots your part touches, and the tests, fakes and fixtures your cards will name.
- Never read another part's code.
  Where your contract names code outside your area, such as an export of a part you depend on or a member it re-signs, look up its signature with `lym quarry glyphs --text <dir>` or `lym quarry expand <glyph>`, and never read that code in full.
- Take the design from the decision record and the seams between parts from the parts file.

## Step 3 — Write your part's cards into `{{.plan_dir}}`

Create `{{.plan_dir}}` if it does not exist, and write your part there as a plan of its own: one `00-overview.md` and one card file per card, numbered from 1, in the format below.

- Write cards for your own part only, and only into `{{.plan_dir}}`: never edit another part's cards, the parts file or the joined plan.
- Create every export your contract names, spelled exactly as the parts file spells it; a `plan:` export is declared on a `**Create:**` bullet with that same handle.
- Use another part's or a shared spot's export by the name the parts file gives it, and only an export of a part you depend on or of a shared spot that names your part; the part check counts those as declared.
- Leave every shared spot to the chair: no card of yours targets a shared spot's path, and a card that needs something from one says what in its `**Intent:**`.
- Leave `## PATTERN changes` out of your overview, since the chair writes it into the joined plan.
- Leave `root:` out of your overview's frontmatter, since the join refuses parts whose `root:` or `language:` differ.
- Number your card files yourself as you write them, keeping each file name, heading and Card Index line in step.
  Never run `lym loom join-parts` or `lym loom insert-card`: both work on the joined plan, which is the chair's.

{{template "loom-block-plan-format"}}

## Step 4 — Write `{{.overview_path}}` last, then check your part

Write `{{.overview_path}}` only after every card file of your part exists.
Then run `lym loom validate-plan --part {{.part_name}}` and fix every finding until it passes.
It checks your part as the plan gate checks a plan, counts the exports of the parts you depend on and of the shared spots that name your part as declared, refuses a card that targets a shared spot, and refuses an export of your contract that no card of yours creates, re-signs or deletes.
Your output existing with that check passed is what tells the chair your part is ready.

Two findings are not yours to fix in your cards; report each to the chair as Step 5 says, and wait for its ruling:
- a `part-export-missing` finding saying the check rewrote an export's handle to another spelling: the contract's spelling must change, which only the chair rules on;
- a refusal saying the parts file is gone or malformed.

## Step 5 — When a contract is wrong

If your contract is wrong or cannot be met, such as an export another part needs that your area cannot create, or a dependency's export that does not fit your cards, message the chair naming the contract line and what you propose, then end your turn and wait for its ruling.
Never change a contract on your own: act on the chair's ruling, and on a ruling it sends you about another part's contract.

## Step 6 — After your part is ready

Keep answering the chair until the step ends.
When the chair asks you to fix your part, change only your own cards, run `lym loom validate-plan --part {{.part_name}}` again until it passes, and then reply to the chair that the fix is done.

## Never use `AskUserQuestion`

Never call the `AskUserQuestion` tool: this session is autonomous, and no operator is present.
````

`contracts/stencils/loom/loom-template-parts-chair.md`, in full:

````markdown
<!-- This is the Parts-Plan-Write chair's opening prompt: the lead of the table internal/loomengine's PartsPlanTable builds, filled by internal/seatengine from the lead's value map.
     plan_dir, overview_path and join_path are seat values; decision_record_path, parts_path, parts_dir and specs_dir are table values;
     parent_directive and edit_directive are rendered by the seat engine, and pattern_directive and friction_directive are optional.
     It includes seat-directive-writer-chair and loom-block-plan-format, each rendered from the same map; the block's gate_tiers and specs_dir come from the table.
     On a fresh start after a plan rotation the seat engine appends the prior-plan block to this prompt. -->

# Parts chair — join the parts into one plan

You are the chair of the part writers: each writer turns one part of the task into plan-format cards.
You check every part against its contract, rule on contracts, join the parts into one plan, write the cards of the shared spots, and record the join.
You write no part of your own.

{{.parent_directive}}
{{.edit_directive}}
{{template "seat-directive-writer-chair"}}
{{.pattern_directive}}
{{.friction_directive}}
## What you read

- Read `{{.decision_record_path}}`, the decision record: the design and scope of the whole task.
  Never read the support log or the board.
- Read `{{.parts_path}}`, the parts file, which you change only to respell an export as Step 1 says, whose format `{{.specs_dir}}/loom/loom-parts-spec.md` defines: each part's contract, and the shared spots no part owns.
- Read each part's cards under `{{.parts_dir}}/<part>/` once its writer's ready line has arrived.
- Read cards, not code.
  Read only the code each shared spot's cards need, and, when you fix a part yourself because its writer no longer answers, only the code of that part's area the fix needs.

## Step 1 — Wait for the parts, and answer the writers

- End your turn until every writer's ready line has arrived.
- A writer may raise a contract it finds wrong.
  Rule on it from the decision record and the parts file, reply to that writer, and tell every other writer the ruling affects, naming the contract line and the ruling.
  Keep every ruling for the join record.
- When a writer reports that its part check rewrote an export's handle to another spelling, change that export in `{{.parts_path}}` to the spelling the check gave, the one edit of the parts file you make, and tell every writer whose part uses the export to use the new spelling.
- When a writer, or your own check, reports that the parts file is gone or malformed, tell your parent, whose way forward is to run the parts step again, and end your turn; you cannot write the parts file.

## Step 2 — Check each part against its contract

For each part, check that its cards deliver the behaviour its contract states and create every export the parts file gives it, spelled as the parts file spells it.

- A part that falls short goes back to its writer: name what is missing or wrong, and wait for the writer's reply that its part check passes again.
- A writer lym told you was stopped no longer answers: fix its part yourself in `{{.parts_dir}}/<part>/`, then run `lym loom validate-plan --part <part>` until it passes.
  A part whose fix you had asked of that writer is still open: check it against its contract again and fix it yourself.
- Go on only once every part stands.

## Step 3 — Join the parts

Run `lym loom join-parts` once every part stands.
It checks each part again and refuses, naming the part to fix, when a part fails its check, leaves an export of its contract uncreated, creates a handle or a path another part also creates, or differs from another part in `language:` or `root:`.
Otherwise it writes every part's cards into `{{.plan_dir}}` as one plan numbered in parts-file order, replacing what was there.
On a refusal, have that part's writer fix it, or fix it yourself when the writer no longer answers, and run `lym loom join-parts` again.
A `lym loom join-parts` that failed with an error, rather than a refusal, wrote no complete plan: run it again.
Once you have written anything into `{{.plan_dir}}` yourself after a join that succeeded, never run `lym loom join-parts` again: a seam you find after that is fixed in the joined plan's cards.

## Step 4 — Write the shared spots and the overview

- Write each shared spot as cards of your own in `{{.plan_dir}}`, each added with `lym loom insert-card`, whose `--help` gives its arguments, and then filled in, in the format below.
  A spot's cards create the exports the parts file gives the spot.
  Place them after every card that makes what the spot uses, and before the first card that uses what the spot creates.
- Insert or remove a card in `{{.plan_dir}}` only with `lym loom insert-card` or `lym loom remove-card`, which renumber the later cards, never by renaming, rewriting under a new name or deleting a card file by hand.
- Rewrite the framing paragraph of `{{.overview_path}}` so it frames the whole task.
- When the decision record has a `## PATTERN changes` section, copy its bullets into the overview's `## PATTERN changes` section, one per entry, without the added-after-Discussion suffix; the plan gate refuses a mismatch.

{{template "loom-block-plan-format"}}

## Step 5 — Check the plan, then write the join record last

Run `lym loom validate-plan`, which takes no arguments, and fix what it reports until it passes.
Then write `{{.join_path}}` last, since its existence ends your step: for each part, whether its cards met its contract and what you sent back or fixed yourself; every ruling you made on a contract; and for each shared spot, the cards you wrote for it and where they went.

## Never use `AskUserQuestion`

Never call the `AskUserQuestion` tool: this session is autonomous, and no operator is present.
````

**ImpactSummary:** New table builder and two new stencils; PlanTable, PartsTable and every existing stencil are unchanged.
