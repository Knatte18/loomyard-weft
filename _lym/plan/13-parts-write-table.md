# Card 13 — parts-write-table

**Create:**
- `contracts/stencils/loom/loom-template-parts.md`
- `contracts/stencils/loom/loom-fragment-parts-light.md`
- `contracts/stencils/loom/loom-fragment-parts-medium.md`
- `contracts/stencils#LoomTemplateParts`
- `contracts/stencils#LoomFragmentPartsLight`
- `contracts/stencils#LoomFragmentPartsMedium`
- `internal/loomengine#partsRole`
- `internal/loomengine#PartsTable`
- `internal/loomengine#TestPartsTable`

**Extends:**
- `contracts/stencils#entries`

**Edit:**
- `contracts/stencils#TestStencils_TopLevelMarkers`

**Uses:**
- `internal/partsparser#File`
- `internal/partsparser#Dir`
- `internal/loomengine#PlanTable`
- `internal/loomengine#TestPlanTable`

**Intent:** Build the `Parts-Write` row's one-seat table (D4) and ship its stencil (D10) with its two reading-depth fragments.

**Table.** `PartsTable` mirrors `PlanTable`: one lead, role prefix `parts` (`partsRole`), segment plan, timeout `cfg.PlanTimeoutMin` minutes, model `cfg.Parts` resolved through `resolveModelChoice` with `ModelKey{Module: "tapestry", Key: "parts"}`, skills `planSkills`, no forks, output `partsparser.File(anchor)`, inputs the decision record and `relevant.md`.
It renders the PATTERN directive as `PlanTable` does, and the friction directive for the note `Parts-Write` in the loom friction directory, a render failure logged and left empty.
It reads the depth fragment at call time: `loom-fragment-parts-light` for `cfg.PartsDepth` `light`, `loom-fragment-parts-medium` for `medium`, through `stencilstore.Read` with `stencil.StripLeadingComment`; an unreadable fragment fails the table, naming the fragment.
Any other depth is a fail-loud wiring guard in `refusal-spec.md`'s sense, an error naming `parts_depth`, which takes no reaching test, since config load already refuses such a value.
Its table values are `decision_record_path`, `relevant_path`, `parts_path` (`partsparser.File`), `parts_dir` (`partsparser.Dir`), `parts_max` (`cfg.PartsMax` as decimal text), `depth_block`, `specs_dir`, `pattern_directive` and `friction_directive`, the last two optional.
The table carries no gate; the producer entry sets it.

`TestPartsTable` (tier0, built as `TestPlanTable` is) covers: the table's role, segment, timeout, output, inputs and model key; `parts: sonnet[low]` reaching the lead's model choice; the composed prompt holding the light fragment's text at depth light and the medium one's at depth medium, and the configured `parts_max`; a stencils directory without the chosen fragment failing the table naming it; and a stencils directory without the friction stencil giving a table whose friction directive is empty.
`TestStencils_TopLevelMarkers` gains `loom-template-parts` with `decision_record_path`, `depth_block`, `edit_directive`, `friction_directive`, `parent_directive`, `parts_dir`, `parts_max`, `parts_path`, `pattern_directive`, `relevant_path` and `specs_dir`, and both fragments with no marker.
The three stencils embed beside `LoomTemplatePlan`, with entries in `entries`.

`contracts/stencils/loom/loom-fragment-parts-light.md`, in full:

````markdown
<!-- This is the light reading depth of the Parts-Write agent, chosen by tapestry.yaml's parts_depth and filled whole into loom-template-parts' depth_block marker by internal/loomengine's PartsTable.
     It declares no marker. -->

Your reading depth is light.
Read the decision record and `relevant.md` as Step 1 says, then run `lym quarry glyphs --text <dir>` over each package directory the decision record names, to see the symbols each package holds.
````

`contracts/stencils/loom/loom-fragment-parts-medium.md`, in full:

````markdown
<!-- This is the medium reading depth of the Parts-Write agent, chosen by tapestry.yaml's parts_depth and filled whole into loom-template-parts' depth_block marker by internal/loomengine's PartsTable.
     It declares no marker. -->

Your reading depth is medium.
Read the decision record and `relevant.md` as Step 1 says, then run `lym quarry glyphs --text <dir>` over each package directory the decision record names, to see the symbols each package holds.
Then run `lym quarry toc <dir>` for each package the task touches, which gives its package doc and each file's header, and `lym quarry expand <glyph>` for each type the decision record names, which gives its signature and members.
````

`contracts/stencils/loom/loom-template-parts.md`, in full:

````markdown
<!-- This is the loom Parts-Write producer's autonomous prompt: one lead seat, built by internal/loomengine's PartsTable and filled by internal/seatengine from the lead's value map.
     parent_directive, edit_directive, decision_record_path, relevant_path, parts_path, parts_dir, parts_max, depth_block and specs_dir are required markers;
     depth_block holds the loom-fragment-parts-light or loom-fragment-parts-medium stencil that tapestry.yaml's parts_depth names.
     pattern_directive and friction_directive are optional and render as nothing when their tier is inactive.
     The parts format's Go contract is contracts/specs/loom-parts-spec.md; this stencil carries its LLM-facing subset. -->

# Parts — split the task into parts with contracts

You are the Parts producer: a single autonomous agent that reads the decision record and splits the task into parts, each one writer's coherent unit, with a contract for each part.
After you, one writer per part turns that part into plan cards, and a chair joins their cards into one plan.
You write contracts, never cards, and you never interview and never ask the operator.

{{.parent_directive}}
{{.edit_directive}}
{{.pattern_directive}}
{{.friction_directive}}
## Step 1 — Read the decision record

Read `{{.decision_record_path}}`.
It is your only source of design and scope: never read the support log or the board.
If it is missing or empty, stop and report that to your parent instead of inventing scope.

Then read `{{.relevant_path}}` if it exists: it lists the files and members the Discussion found relevant, and it never adds scope.

## Step 2 — Read only what your depth allows

{{.depth_block}}

Read no code beyond that.
When you cannot split the task without reading more, read no further: write a parts file of one part that covers the whole task, and say in the split note why it does not split.

## Step 3 — Split the task

- Split by behaviour along module boundaries, so each part is one writer's coherent unit: its writer turns it into cards reading only its own area.
- A part too small to hold a card of its own is too fine a split, and so are two parts whose areas share most of their files; merge them.
- Hold at most {{.parts_max}} parts.
- Order the parts so each comes after every part it depends on, with no cycle.
  A part depends on an earlier part when its cards use what that part creates, re-signs or deletes.
- Name every decision id of the record (`D1`, `D2`, …) in some part or shared spot, and no id the record does not hold.
- A file no single part owns, such as a registry, a config template, `PATTERN.md` or a shared doc, is a shared spot: no part's cards touch it, and the chair writes its cards.

## Step 4 — Write the parts file

Create `{{.parts_dir}}` if it does not exist, and write `{{.parts_path}}`, and nothing else under `{{.parts_dir}}`.
`{{.specs_dir}}/loom/loom-parts-spec.md` is the format's full contract; the subset below is what you write.

```markdown
# Parts: <task title>

## Split note

<how the task splits and why along these lines, or why it does not split>

## Part: <name>

- Decisions: D1, D4
- Takes in: <what the part reads from the decision record, and from each earlier part it depends on>
- Delivers: <the behaviour the part's cards deliver>
- Exports:
  - `plan:internal/x#NewThing`
  - `internal/x/newfile.go`
- Depends on: <earlier part>, <earlier part>
- Area:
  - `internal/x`
  - `contracts/specs/x-spec.md`

## Shared spot: `<path>`

- Decisions: D3
- Needed by: <part>, <part>
- Needs: <for each part listed, what it needs from the spot>
- Exports:
  - `plan:internal/y#Registered`
- Uses:
  - `plan:internal/x#NewThing`
```

- Write the split note in every parts file: how the task splits, or why it does not.
- One `## Part: <name>` section per part, each after every part it depends on, then one `## Shared spot: `<path>`` section per shared spot.
- A part name is lower case, starts with a letter and holds only letters, digits and `-`; it names the part's directory and its writer, so it never begins with `archive-`, and no two parts share one.
- A part needs `Decisions`, `Takes in`, `Delivers` and `Area`; leave `Exports` and `Depends on` out when the part has none, never writing `none`.
- `Exports` lists, one backticked ref per line, what another part or a shared spot uses from this part: each `plan:` handle and new file path the part's cards create, and each existing member they re-sign or delete, spelled as `lym quarry` gives it.
  A `plan:` handle is `plan:<package directory>#<Name>`, and the part's writer declares it with exactly that spelling.
- `Area` lists the packages, as directory paths, and the files the part's writer may read and change.
  A shared spot's path is never a file entry of a part's area, though it may sit inside a package an area names.
- A shared spot needs `Needed by` and `Needs`; list `Decisions`, `Exports` and `Uses` when it has them.
  `Uses` lists the part exports the spot's cards use, each from the first part `Needed by` names or an earlier one.
- No ref is exported twice, by two parts, two shared spots or one of each.

## Step 5 — Self-check before ending your turn

Run `lym loom validate-parts`, which takes no arguments, and fix every finding it reports; run it again until it passes.
Your turn ends once the parts file passes.

## Never use `AskUserQuestion`

Never call the `AskUserQuestion` tool: this session is autonomous, and no operator is present.
````

**ImpactSummary:** New table builder and three new stencils; PlanTable and every existing stencil are unchanged.
