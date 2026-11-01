```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end on one concrete case and checked the end state against the decision's purpose.

**Walkthrough list**

- **D1**: `plan_split` true, Drift-Baseline Done. Parts-Write runs, then Parts-Plan-Write; its Done enters Plan-Write, whose gate skips it to Plan-Bouncer. A later goto onto Plan-Write writes a Done history entry, so the refusal names Plan-Write. **Purpose met.**
- **D2**: a profile sets `parts_depth: medium` and the `part_writer` model. The table source reads both keys into the seat model and the stencil block. **Purpose met.** I did not check the enum refusal; it is out of this lens.
- **D3**: a parts file with A, then B depending on A, and a spot S needed by A and B that uses an export of A. The check admits it, and S's cards go between A's cards. **Purpose met.**
- **D4**: a fresh Parts-Write after an earlier Parts-Plan-Write. The prep moves `parts.md`, the part dirs and `join.md` into an archive. `archiveStaleOutputs` then finds nothing. A gate-exhausted Stuck returns a non-empty pointer. **Purpose met on Done; Stuck is unclear (finding 3).**
- **D5**: writer A ends done and accepted, the chair asks it for a fix, and the machine restarts while A is mid-fix. Resume with a live chair: the probe removes A's kept pane, and Accept runs on a half-edited part. **Purpose missed (finding 1).**
- **D6**: writer B exhausts its `plan-part` gate. `watch` ends the step and `join` stops the chair. With the stated EndedStep mapping this becomes Stuck with B's overview as pointer, and the decorator commits `_lym/parts/`. **Met for the gate/Accept path.** For writer died or never started: **purpose missed (finding 2).**
- **D7**: part B's card uses `plan:` handle X exported by A. `--part B` counts X as declared, and the card stays off the spot path. **Purpose met.**
- **D8**: two parts, A with 3 cards and B with 2. The join gives cards 1–5, merges the Index, rewrites `Commit:` prefixes, and refuses an unmet export. Dependencies come from `Uses:` against order, so no authored dependency field is needed. **Purpose met.**
- **D9**: chair waits for done lines, then join, insert-card for S, framing and PATTERN changes, validate-plan, then `join.md`. The row's `plan` gate runs over `_lym/plan`. The chair's inputs list only the part overviews; inputs are prompt text, never stat'ed at spawn. **Purpose met.**
- **D10–D12**: stencil includes and markers walked as stated. **Purpose met.**
- **D13**: phase mapping and reinvoke policy. Reinvoke leads to the D5 resume path, which hits finding 1. **Otherwise met.**

**Findings**

1. **MEDIUM** · design · D5 resume paragraph (decision-record.md:118-120) and D6 Bound (:147)
   **Summary:** a resume that interrupts a fix round of a done writer either loses the fix silently or ends the step and redoes every part.
   **Evidence:**
   - A done writer's run record is terminal, so the probe "removes" its kept pane and reports it not found (resume.go, `Probe` doc).
   - D5 then runs Accept on whatever the half-applied fix left on disk.
   - If Accept passes, the chair is told the part "stands", even though it asked for a fix that is only partly applied. D9's "chair fixes it" covers this only if the chair links the two.
   - If Accept fails, the step ends, the run goes Stuck, and the rerun is fresh, so every finished part is archived (multillm.go `Call`, which runs `prepareFreshSpawn` on the fresh path).
   - D5's stated purpose, a resume that keeps finished parts, is missed in this case, and neither D5 nor D6's Bound states it.

2. **MEDIUM** · design · D6 (decision-record.md:144-145)
   **Summary:** "a writer that failed to start or died stays an error carrying its start error or wrapping `ErrAgentDied`" describes a mapping today's code does not produce.
   **Evidence:**
   - When a required writer dies, `watch` calls `ender.end`, and `join` stops the chair (engine.go `watch`, `join`).
   - `mapOutcome` reads only `result.Chair`, never `SeatResult.StartError` or `EndedStep` (multillm.go `mapOutcome`).
   - So the error carries the stopped chair's outcome, not the writer's start error.
   - "Stays" implies no change is needed, but D6 itself says today's path gives "a reason naming the chair".
   - The record leaves this mapping, for both the died case and the not-started case, unstated.

3. **LOW** · consistency · D4 (decision-record.md:93) and the D4 acceptance criterion (:280)
   **Summary:** D4's decorator "commits `_lym/parts/` on Done", while the decorator it copies commits on any non-empty pointer, so it is unclear whether Parts-Write commits on a Stuck.
   **Evidence:**
   - `NewPlanWrite` commits on a gate-failed Stuck as well as on Done (planwrite.go:59-63).
   - D6 states "on Done and on Stuck alike".
   - Read literally, D4 leaves a gate-exhausted Parts-Write halt with uncommitted moves and `parts.md`, a dirty tree.
   - D4's acceptance criterion covers Done only.

**Tool calls**

1. Grepped the tapestry recipe for the Drift-Baseline and Plan-Write rows: the context lacked the recipe rows. The first attempt used a wrong path.
2. Located the tapestry recipe and plan spec files: the first attempt found no recipe.
3. Read the Drift-Baseline and Plan-Write rows and the plan spec's dependency rules: needed to check D1's edges and how D8 joins dependencies.
4. Grepped the plan spec for `first_card` and the overview section rules: needed to check that D7 and D8's part overviews meet the format.
5. Grepped for `first_card` and rework routing: needed to check whether D8's numbering from 1 conflicts with a rework generation. It does not, since rework does not go through the split rows.
6. Grepped seat input handling: needed to check whether D6's chair inputs, the part overviews, must exist when the chair spawns. They need not; inputs are prompt text.
