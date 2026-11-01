```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: each decision's mechanism run on one concrete case, with the end state checked against the decision's purpose.

**Findings**

1. **MEDIUM — D11/D12: the part writers and the chair never get the quarry glyph rule.**
   - The rule "copy every glyph from a `lym quarry` answer, never spell one" sits in `loom-template-plan.md` Step 2 ("Look up glyphs with `lym quarry`"), outside the Step 3 card-format sections that D12 moves into `loom-block-plan-format`.
   - D11's stencil bullets and D9's chair steps never mention quarry.
   - Case: a part writer writes an `Edit` target. Its prompt holds the card format but not the hard rule, and the block's own text still says "the glyph spelling rules Step 3 spells out", a step number the part-writer and chair stencils do not have.
   - Effect: glyph findings from `--part` and the join gate that the shared block was meant to prevent.
   - Fix: put the quarry subsection in the block, or have both new stencils include it, and drop the step-number cross-references from the block.

2. **MEDIUM — D3/D7/D11: the area/shared-spot rule fires on the commonest shared spots.**
   - D3 refuses a shared-spot path "inside a part's area", and an area is "the packages and files its writer may read and change".
   - Case: part `producers` owns `internal/shedrecipe` (it adds two Constructor entries), and the registry file in that package is the shared spot.
   - Effect: the parts check refuses the file, so `Parts-Write` must either drop the package from the area or not name the shared spot. Dropping it means the writer may not read `internal/shedrecipe` at all, which conflicts with D11's "code of its own area and of the shared spots it touches".
   - Fix: let an area hold a package while a shared spot names a file inside it, refusing only an exact path held by both, and have the D7 card-target check enforce that.

3. **MEDIUM — D7: a part cannot use a name that only a shared spot creates.**
   - Exports exist only per part, and shared spots carry no exports.
   - Case: the chair's shared-spot card adds a field to the config struct, and a part card puts it under `Uses` as a `plan:` handle. The `--part` check flags `handle-dangling`, since no dependency exports it. The writer can neither create it (shared-spot target, refused) nor drop it.
   - Effect: the gate runs out of attempts, and a required writer ends the step.
   - Fix: let shared spots declare the names they create, and have the part check count those as declared, as it does for dependency exports.

4. **LOW — D6: the chair's inputs are not stated.**
   - The table validation requires "every advisor output is among the lead's inputs".
   - D6 names the writers' inputs and the chair's output, but not that the chair's inputs must list each part's `00-overview.md`, plus the parts file and record the chair reads.
   - The engine would refuse the table at validation if this is missed. One clause in D6 fixes it.

5. **LOW — D4/D6: the archive steps overlap with what the producer already does.**
   - `MultiLLMProducer.Call` already runs `archiveStaleOutputs(table.Outputs())` after `prepareFreshSpawn`, which archives each seat's output file.
   - D4's archive of the whole `_lym/parts/` and D6's archive of the part directories are still needed: stale card files would otherwise trip `index-file-mismatch` orphan findings.
   - The record could say the preparation archives what the output-file archive misses (card files beside a part overview), so the plan writer does not duplicate or drop either step.

6. **LOW — D12: part overviews may end up with a `## PATTERN changes` section.**
   - The shared block's overview subsection tells the writer that the overview carries the record's `## PATTERN changes`.
   - Part writers following it will write that section into their part overview. D8 does not carry it over, and D7 skips `pattern-changes-mismatch`.
   - Harmless, but wasted turns; the part-writer stencil could say a part overview omits it.

**Decisions walked**

- **D1** — Case: flag off and drift off. Drift-Baseline is skipped to Parts-Write, which is skipped to Parts-Plan-Write, which is skipped to Plan-Write; Plan-Write's gate reads `plan_split != true`, which holds, so it runs. Flag on: Parts-Plan-Write's Done enters Plan-Write, whose gate does not hold, so it skips to Plan-Bouncer. `tapestry` is confirmed as the settings module the gate reads (recipe `tapestry.plan_reviewer_fan`, keymove). **Purpose met.**
- **D2** — Case: a profile sets `tapestry.plan_split: true` and `parts_depth: medium`; configreg checks the keys and Seed-Child writes them. **Met.**
- **D3** — Case: three parts, where C depends on A and the registry is a shared spot inside B's package. Order and coverage checks pass; the area check refuses the file. **Partly met** (finding 2).
- **D4** — Case: a goto reruns Parts-Write over an existing `parts.md`. The preparation archives the directory, the output-file archive is then a no-op, the new session writes the file, and the gate and commit follow. **Met** (finding 5 is a wording overlap).
- **D5** — Case: two writers. The chair ends its turn and its hold is deferred while the writers are mid-turn. Writer A's accepted done queues its line and wakes the chair. Writer B dies; as a required seat this ends the step. **Met.**
- **D6** — Case: the source reads `parts.md` before the probe; the probe finds no live chair; the preparation rotates the plan directory and the part directories, archives the outputs, and the table runs. Chair inputs are unstated (finding 4). **Met, with a gap.**
- **D7** — Case: part B uses part A's exported handle and passes; part B uses a shared-spot name and is stuck. **Partly met** (finding 3).
- **D8** — Case: two parts of three cards each. Each part is checked and renumbered on its own, to 1–3 and 4–6; the overview is merged. **Met.** Insert-card's precondition ("after the last card webster has begun") holds before Webster runs.
- **D9** — Case: the chair joins, inserts a shared-spot card at position 4 with `insert-card` (which leaves a stub the chair then fills), validates, and writes `join.md` last; the chair is done and the `plan` gate runs on `_lym/plan`. **Met**, but the chair's glyph spelling is unguided (finding 1).
- **D10** — Case: a light-depth split falls back to one part. A table of chair plus one writer is two seats, so the writer-chair directive applies. **Met.**
- **D11** — Case: a writer writes cards using quarry glyphs, but no quarry rule is in its prompt. **Not met** (finding 1).
- **D12** — Case: the block moves out of Plan-Write's Step 3, and Plan-Write's prompt stays byte-identical as long as the block keeps `{{.plan_dir}}` and its other markers. In the part writer, seat values override `plan_dir` with the part directory. **Met for Plan-Write; the new stencils inherit gaps** (findings 1 and 6).
- **D13** — Case: `lym report timing` maps both row families to the plan phase; interrupt reinvoke goes through Probe and Resume. **Met.**

**Tool calls**

1. `lym loom insert-card --help`: whether insert-card fits the chair's step and its preconditions. Context had only its renumbering, not its CLI bounds.
2. `validateDashed` in agentname: the role grammar for `parts-plan-&lt;part&gt;` names. Context showed only that `ValidateRole` delegates to it.
3. `shedadapters/multillm.go` `Call`: the order of table source, probe, preparation and output archive. Context had only the constructor signature.
4. Tapestry recipe `_setting` lines, configreg keymove and the plan template's Step 2: whether `tapestry.plan_split` names a real module, and where the quarry rule sits. Context had the template's headings, not the Step 2 text.
