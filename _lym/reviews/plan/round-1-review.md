---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: handler
    location: _lym/plan/16-parts-producers.md (Env gains PartsMax int)
    summary: >-
      Card 16 carries tapestry.yaml's parts_max as a field of its own on shedrecipe.Env, which PATTERN-shed-recipe-registry forbids for a row's value
    fix: >-
      In card 16, drop the `PartsMax int` bullet from **Env** and every later mention of `PartsMax`
      (the `parts` gate requirement, the `loomCLI.wire` sentence "`PartsMax` is the loaded `parts_max`",
      the `envkit.FullEnv` sentence "and `PartsMax` with 6", and the `TestResolveGateSpec_Rejections` row "a `PartsMax` of 0").
      Rewrite the **Gate vocabulary** sentence so `resolvePlainGateClosure`'s `parts` case requires `AnchorPath` and `DecisionRecordPath`
      absolute and a non-nil `Setting` seam, reads the value with `env.Setting("tapestry", "parts_max")`, refuses a value that is not
      an integer of 1 or more with an error naming `tapestry.parts_max`, and returns `loomshed.NewPartsGate(env.AnchorPath, env.DecisionRecordPath, max)`.
      Replace the `PartsMax` rejection row with rows for a nil `Setting` and for a `Setting` answering 0, each refused naming its source.
      Change the **Env** lead-in from "five seams" to four, and the ImpactSummary's "five Env seams" to four.
  - id: F2
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/plan/12-plan-format-block.md (The template) and 00-overview.md Shared Decision plan-prompt-keeps-every-line
    summary: >-
      Card 12 reorders Plan-Write's composed prompt, moving the glyph subsection from Step 2 into Step 3 and two sentences after the block,
      while the parent's added decision approves the move only on condition that Plan-Write's composed prompt stays the same
    fix: >-
      Ask the parent, through `lym creel post --parent --expect-reply`, to rule on the Shared Decision plan-prompt-keeps-every-line:
      either it records an Added-after-Discussion entry accepting that the glyph subsection now opens Step 3's card format and the two
      kept sentences follow the block, which the card then cites in its Intent, or it names a different arrangement, such as a second
      included block for the glyph subsection, and card 12 is rewritten to it.
      Until a recorded decision covers the reorder, card 12 must not ship a Plan-Write prompt whose line order differs from today's.
  - id: F3
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/plan/07-loomshed-part-checks.md (ValidatePart part-export-missing) and _lym/plan/14-parts-plan-write-table.md (chair Step 1 and What you read; writer Step 4)
    summary: >-
      The plan lets the chair edit the parts file to respell an export and adds a part-export-missing check to the part check,
      and the decision record licenses neither, since D7 lists the part check's only differences and D3 makes the parts file Parts-Write's output
    fix: >-
      Drop the chair's parts-file edit. In card 7, have `ValidatePart` and `planparser.JoinParts` compare each contract export with the
      part's created, re-signed and deleted refs by the canonical spelling the resolve pass gives, so a respelled handle still matches.
      Then remove the respell clause from `part-export-missing`'s way forward.
      In card 14's chair stencil:
      remove the bullet "When a writer reports that its part check rewrote an export's handle to another spelling …",
      and change "which you change only to respell an export as Step 1 says" to "which you never change".
      In the part writer stencil, remove the first bullet under "Two findings are not yours to fix in your cards"
      and reword that lead-in to one finding.
      Keep `part-export-missing` in `ValidatePart` only if the parent records it in an Added-after-Discussion entry as a fourth D7 difference;
      otherwise drop it from card 7, card 14's Step 4 sentence and card 18's tests, and leave the unmet export to the join, as D8 says.
  - id: F4
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/12-plan-format-block.md (Tests)
    summary: >-
      Card 12 plans permanent assertions on stencil static prose, a committed golden of Plan-Write's whole composed prompt and a check that
      the block's text holds no Step, which the rubric's test-economy rule flags
    fix: >-
      In card 12, drop the Create target `internal/loomengine/testdata/plan-prompt.golden` and the sentence
      "The card writes that golden file first, from the unmoved stencil, then moves the text."
      Replace the `TestPlanTable` golden assertion with a one-time check the card runs while it moves the text:
      compose the lead's prompt with `seatengine.Prompts` before and after the move and compare the two as line multisets, with nothing committed.
      Drop the assertion that the block's registered default holds no `Step `.
      Keep the `TestStencils_TopLevelMarkers` row, which pins the block's markers.
  - id: F5
    severity: MEDIUM
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/12-plan-format-block.md (Edit list)
    summary: >-
      Card 12 moves the card format out of loom-template-plan.md but leaves every spec, doc and rubric line that says the format is pinned in that stencil
    fix: >-
      Add to card 12's **Edit:** list `contracts/specs/loom-plan-spec.md`, `contracts/specs/webster-spec.md`, `docs/overview.md` and
      `contracts/stencils/loom/loom-rubric-plan-review.md`.
      Add an Intent paragraph that rewords each pointer to name `loom-block-plan-format`, which `loom-template-plan` includes, as the home of the card format:
      `loom-plan-spec.md` lines 3, 7 and 831, `webster-spec.md` lines 14 and 73, and `docs/overview.md` line 422.
      In the plan-review rubric's line 121, change only the pointer, so it names both the template and the block it includes, with no other rubric change.
  - id: F6
    severity: MEDIUM
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/07-loomshed-part-checks.md (no spec target)
    summary: >-
      The plan spec's list of modes that run the loomshed context checks, at loom-plan-spec.md line 416, and its check list go stale
      once ValidatePart clears DecisionRecordPath and adds part-export-missing, and no card edits them
    fix: >-
      Add `contracts/specs/loom-plan-spec.md` under card 7's **Extends:**.
      Add an Intent sentence saying the spec's mode list at line 416 gains `--part`, the `plan-part` gate and the writer seat's Accept,
      which run the context checks without the record, so `pattern-changes-mismatch` and `pattern-line-unprescribed` are left to the joined plan's `plan` gate.
      If `part-export-missing` stays (see F3), the sentence also adds its row to the validation checks.
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/17-parts-recipe-rows.md (internal/reportengine/doc.go under Uses)
    summary: >-
      internal/reportengine/doc.go line 130 lists the plan phase's rows as every Plan row, Drift-Baseline and Drift-Await,
      which card 17 makes false by mapping Parts-Write and Parts-Plan-Write to PhasePlan
    fix: >-
      Move `internal/reportengine/doc.go` from card 17's **Uses:** to its **Edit:** list,
      and add an Intent sentence saying the doc's plan-phase list names `Parts-Write` and `Parts-Plan-Write`.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/05-planglyph-part-scope.md (internal/planglyph/doc.go under Uses)
    summary: >-
      internal/planglyph/doc.go lines 59 and 85 say caller-uncovered is reached at the plan gates, which stops holding for a part scope, and card 5 leaves the doc unedited
    fix: >-
      Move `internal/planglyph/doc.go` from card 5's **Uses:** to its **Edit:** list,
      and add an Intent sentence saying the doc states both part-scope exemptions:
      `resolvePass` treats `Part.Paths` as forthcoming, and `planGatePass` reports no `caller-uncovered` when `plan.Part` is set.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/09-seat-writer-kind.md (doc.go and symbol docs)
    summary: >-
      Card 9 adds a Writers section to seatengine's doc.go but leaves the existing sentences that writers falsify,
      at doc.go lines 26, 61 and 87 and in the Seat.Accept and Table.Validate doc comments
    fix: >-
      In card 9's Intent, add that `internal/seatengine/doc.go` lines 26, 61 and 87 are reworded for writer seats:
      a writer table's lead includes `seat-directive-writer-chair`, a writer can end the step before the chair, and a writer gets no up line.
      Add the same for the `Seat.Required` and `Seat.Accept` doc comments, so Accept belongs to a seat that ends the step,
      and for the `Table.Validate` doc comment's lead-directive rule.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/06-planparser-join.md (joined overview Shared Decisions) and _lym/plan/14-parts-plan-write-table.md (chair Step 4)
    summary: >-
      The join copies each part's Shared Decisions body verbatim, so every Applies to line keeps the part's own card numbers,
      which no longer match after renumbering, and the chair stencil never asks for them to be fixed
    fix: >-
      In card 14's chair stencil Step 4, add a bullet after "Rewrite the framing paragraph …":
      "In `## Shared Decisions`, rewrite each carried decision's `Applies to` line to the joined plan's card numbers, which `lym loom join-parts` printed for each part's cards."
  - id: F11
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/06-planparser-join.md (unmet-export refusal) and _lym/plan/19-join-parts-verb.md (item 3)
    summary: >-
      planparser.JoinParts' unmet-export refusal cannot be reached through lym loom join-parts, because loomshed.JoinParts first refuses any part whose ValidatePart reports part-export-missing
    fix: >-
      If F3 keeps `part-export-missing` in `ValidatePart`, reword card 19's item 3 so an unmet export reaches the verb as a failing-part refusal carrying `part-export-missing`,
      and say in card 7's `JoinParts` paragraph that `planparser.JoinParts`' own unmet-export refusal is the format writer's backstop, which card 6's `TestJoinParts` reaches.
      If F3 drops `part-export-missing`, no change is needed.
  - id: F12
    severity: LOW
    class: design
    origin: handler
    location: _lym/plan/18-validate-parts-verbs.md (TestValidatePlanCmd_Part)
    summary: >-
      TestValidatePlanCmd_Part plans handle-dangling and part-spot-target rows that card 7's TestValidatePart already covers over the same ValidatePart
    fix: >-
      In card 18, drop the row "a card targeting a spot failing with `part-spot-target`" from `TestValidatePlanCmd_Part`.
      Keep one failing row, the dangling handle, for the findings envelope and exit code, beside the passing, unknown-part and absent-parts-file rows.
  - id: F13
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/18-validate-parts-verbs.md (validate-plan --part)
    summary: >-
      validate-plan --part refuses off-recipe through the PlanWrite row rather than a Parts-Plan-Write row,
      so a run that holds no parts rows gets the absent-parts-file error, whose way forward points at a Parts-Write row
    fix: >-
      In card 18's **validate-plan --part** paragraph, replace "It adds no recipe check of its own …" with a check made before the parts file is read.
      With `--part` set, `validatePlanCmd` refuses unless `loomrecipe.HasEngine(recipe, "PartsPlanWrite")`, with the message
      "loom: validate-plan --part does not apply to this run, whose %q recipe has no Parts-Plan-Write row and so no parts; way forward: run \"lym loom validate-plan\" without --part".
      Add that refusal as a row of `TestValidatePlanCmd_Part`.
---

### [BLOCKING] Env carries parts_max as a field of its own

**Location:** `_lym/plan/16-parts-producers.md`, **Env** and **Gate vocabulary** paragraphs.

**Issue:** `PATTERN-shed-recipe-registry` says a recipe row's value is looked up by key through `Env.Setting`, never carried as a field of its own on the recipe's `Env`, and that a row never gets its own `Env` field.
Card 16 adds `PartsMax int` to `shedrecipe.Env`, filled by `loomCLI.wire` from the loaded `parts_max` and read only by the `parts` gate of the `Parts-Write` row.
That is a plain config value, unlike the per-producer table and commit seams beside it, which follow the established `PlanTable`/`CommitPlan` convention in `internal/shedrecipe/recipe.go`.
The decision record's Constraints name `PATTERN-shed-recipe-registry` for these rows: "each value looked up by key".

### [MEDIUM] Plan-Write's composed prompt changes order against the parent's condition

**Location:** `_lym/plan/12-plan-format-block.md` **The template**, and `00-overview.md` Shared Decision `plan-prompt-keeps-every-line`.

**Issue:** The first Added-after-Discussion entry approves moving the card format and glyph sections into `loom-block-plan-format` "on condition that Plan-Write's composed prompt stays the same", and D12 limits any change to "cross-references the move rewrites".
Card 12 moves the glyph subsection from Step 2 (today's lines 54–70 of `contracts/stencils/loom/loom-template-plan.md`) to the head of Step 3's card format.
It also moves the insert-card and PATTERN-changes sentences out of their sections to after the block.
The Shared Decision redefines "the same" as "every line, in any order".
The constraint the plan resolves is real: `internal/seatengine#validateStencil` includes a block whole and one level deep.
Picking an interpretation of the parent's explicit condition is still the parent's call, not the plan's.

### [MEDIUM] The chair edits the parts file, and the part check gains a check D7 does not list

**Location:** `_lym/plan/07-loomshed-part-checks.md` (`ValidatePart`'s `part-export-missing` and its way forward); `_lym/plan/14-parts-plan-write-table.md` (chair stencil "What you read" and Step 1; part writer stencil Step 4).

**Issue:** D7 defines the part check as the no-flag set with exactly three differences; card 7 adds a fourth blocking check, `part-export-missing`.
Its way forward has the chair "rule the contract's export to the canonical spelling in the parts file", and card 14's chair stencil makes that an edit of `parts.md` by the chair.
The decision record gives the parts file to `Parts-Write` alone (D3, D4).
D9's rulings are messages to writers, not edits of the contract file.
The guard-reach lens also traced that this edit is what makes `ValidatePart`'s and `PartsPlanTable`'s malformed-parts-file refusals reachable mid-row.
The added-decision entry on the stencils asks that nothing contradict another stencil.
The `Parts-Write` stencil tells its agent the parts file is its own product, and the chair stencil now rewrites it.

### [MEDIUM] Card 12 pins Plan-Write's whole prompt in a committed golden

**Location:** `_lym/plan/12-plan-format-block.md` **Tests**.

**Issue:** The rubric's test-economy bullet flags "a card that plans an assertion on a stencil's static prose".
`internal/loomengine/testdata/plan-prompt.golden` is the whole composed Plan-Write prompt, kept as a permanent assertion, so every later wording change to `loom-template-plan.md` or the block forces a golden update.
The "holds no `Step `" assertion checks the block's prose directly.
The acceptance criterion that the composed prompt keeps its content is a property of this card's move, checked once while the card runs.

### [MEDIUM] Docs that say the card format lives in loom-template-plan.md go stale

**Location:** `_lym/plan/12-plan-format-block.md` **Edit:** list.

**Issue:** Verified at `contracts/specs/loom-plan-spec.md` lines 3, 7 and 831, `contracts/specs/webster-spec.md` line 14, and `docs/overview.md` line 422.
Each says the LLM-facing plan format is pinned in `contracts/stencils/loom/loom-template-plan.md`.
`contracts/stencils/loom/loom-rubric-plan-review.md` line 121 is the reviewer's pointer to "the plan writer's own stencil".
After the move, that file holds only `{{template "loom-block-plan-format"}}` where the rules were, and no card edits any of these lines.

### [MEDIUM] The plan spec's mode list misses the part check

**Location:** `_lym/plan/07-loomshed-part-checks.md`.

**Issue:** `contracts/specs/loom-plan-spec.md` line 416 states which modes run the loomshed context checks.
Card 7's `ValidatePart` runs them with `DecisionRecordPath` cleared and adds `part-export-missing`.
Card 4 extends the spec only with the part-scope field and `part-spot-target`, and card 7 lists no spec target.

### [LOW] reportengine's doc misses the two plan-phase rows

**Location:** `_lym/plan/17-parts-recipe-rows.md`.

**Issue:** Verified at `internal/reportengine/doc.go` line 130: "plan every Plan row, Drift-Baseline and Drift-Await".
Card 17 maps both new rows to `PhasePlan` and lists the doc only under Uses.

### [LOW] planglyph's doc misses the part-scope exemptions

**Location:** `_lym/plan/05-planglyph-part-scope.md`.

**Issue:** Card 5 changes when `caller-uncovered` and `glyph-not-found` fire, and lists `internal/planglyph/doc.go` only under Uses.

### [LOW] seatengine's existing doc sentences contradict writer seats

**Location:** `_lym/plan/09-seat-writer-kind.md`.

**Issue:** Verified at `internal/seatengine/doc.go` line 26 ("The lead's stencil of a table of more than one seat includes seat-directive-chair") and line 87 ("Only a required advisor can end the step before the chair").
The `Seat.Required` and `Seat.Accept` comments at `internal/seatengine/table.go` lines 86–90 say Accept belongs to a Required advisor only.
Card 9 changes each rule but prescribes only a new `# Writers` section.

### [LOW] Joined Shared Decisions keep stale card numbers

**Location:** `_lym/plan/06-planparser-join.md`, `_lym/plan/14-parts-plan-write-table.md`.

**Issue:** `planparser` keeps `## Shared Decisions` as raw text (`internal/planparser/plan.go` line 89), so nothing rewrites the part-local "Applies to: cards N" lines.
Plan-Review and Webster then read decisions pointing at the wrong cards.

### [LOW] The join's own unmet-export refusal is unreachable through the verb

**Location:** `_lym/plan/06-planparser-join.md`, `_lym/plan/07-loomshed-part-checks.md`, `_lym/plan/19-join-parts-verb.md`.

**Issue:** `loomshed.JoinParts` refuses a part failing `ValidatePart`, whose `part-export-missing` applies "the same test `planparser.JoinParts` applies", before calling the planparser join.
The planparser refusal is reached only by card 6's direct unit test.
Card 19's row that passes "an unmet export" through rests on a fake `join`.

### [LOW] TestValidatePlanCmd_Part repeats ValidatePart's rows

**Location:** `_lym/plan/18-validate-parts-verbs.md`.

**Issue:** `TestValidatePart` (card 7) already covers `handle-dangling` and `part-spot-target` over `ValidatePart`.
The verb test needs one failing row to pin the envelope, not one row per finding kind.

### [LOW] validate-plan --part has no Parts-Plan-Write row check

**Location:** `_lym/plan/18-validate-parts-verbs.md`.

**Issue:** The decision record's PATTERN-loom-contract constraint has `--part` refuse by the recipe's rows.
Card 18 leans on the `PlanWrite` row instead, which holds today only because tapestry is the one recipe with that row (`contracts/recipes/tapestry-recipe.yaml`).
The fallback refusal is `ValidatePart`'s absent-parts-file error, whose way forward is `lym loom goto --to Parts-Write`.
That way forward is wrong for any recipe with a `Plan-Write` row and no parts rows.

## Focus questions

- Fidelity: every Decision D1–D13 and each Added-after-Discussion entry has a carrying card.
  - D1: cards 17 and 20. D2: cards 1, 13, 14 and 16. D3: cards 2, 3, 7 and 18. D4: cards 13, 15 and 16. D5: cards 8, 9 and 10. D6: cards 10, 11 and 14–16. D7: cards 4, 5, 7 and 18. D8: cards 6, 7 and 19. D9 to D12: cards 12–14. D13: cards 13, 14 and 17.
  - The added decisions are carried as well: signature lookups (card 14, writer Step 2), the chair reading one part's code (card 14) and `LYM_STRAND_NAME` (card 19).
  - The departures are F2 and F3.
- Card order: no card uses a symbol only a later card creates. Cards 9–11 build only on card 8 and on each other, and cards 13–15 use only earlier cards.
- Attack surface and refusal reach:
  - Card 19 states its bound, and card 18 states that `validate-parts` is read-only.
  - Each refusal of cards 7, 18 and 19 has a reaching input and test, except F11.
  - Card 20 adds no new path.
- Glyphs and arrows on cards 4, 5, 6, 9, 10, 11 and 20: member glyphs are used where the Intent names symbols. Card 20 carries the arrows for both re-signed functions, and no other listed card changes a signature.
- Test economy and Verify lines: no card has a `**Verify:**` line, and the test-economy findings are F4 and F12.
- Research steps:
  - The effective plan timeout source is settled. `internal/loomcli/wiring.go` already loads shuttle's config (line 425), which card 16 tells `PartsPlanTable`.
  - The one-level include limit is researched and cited (card 12).
  - No card leaves a fact to be found while it runs.

The combined `lym loom validate-plan --overlay` run of the target changes in F4, F5, F6, F7 and F8 reported no blocking finding beyond the empty-overlay baseline, which had none.
