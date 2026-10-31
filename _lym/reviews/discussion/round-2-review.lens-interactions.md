```yaml
lens: interactions
outcome: reported
```

Task: run the interactions lens. That means finding every pair of decisions that act on the same state, checking whether both still hold, and checking the acceptance criteria and constraints against the decisions they touch.

**Pairs checked**

- D1 × the Added-after-Discussion build-segment entry: held. Tapestry's Webster row has engine `Implementer`, so "exactly one Implementer or DarnWrite row" matches "Plan-Write, its perch and Webster".
- D1 × D4 (the record row comes before the build segment on darn's `on_done` path: Preflight → Record-Write → Drift → Darn-Write): held.
- D1 × D8 (`verify_source` is required and read by the wiring): held.
- D1 × D10 (Preflight runs the `batcher.yaml` load when the recipe has an Implementer row; tapestry's Webster is one): held.
- D2 × D11 (a `beam` seed is refused as a non-loom recipe; the pawl exception does not admit it): held.
- D2 × D3 (peers resolves a pawl peer from its project entry): held.
- D2 × D7 (pawl's snapshot is written by its own path, `pawlwiring.go:135`, not by the darn commit): held.
- D2 × D8 (pawl keeps its verify source under the exception): held.
- D2 × D13 (the override move applies to pawl too, and Scope names it): held.
- D2 × D14 (the non-loom refusal names `lym shed` verbs, which stay): held.
- D3 × D4 (`decision add --pattern-*` on a Record-Write record, and the fence-aware PATTERN insert): held.
- D3 × D5 (the history check keeps appended PATTERN bullets): held.
- D4 × D5 (the unified validator on a mechanical record; no relevant list and no citation check for a RecordWrite row): held.
- D4 × D5 × D6 (a record the validate-only gate refuses, when the writer may not edit it): broken (F3).
- D4 × D6 (the missing-record refusal and the stop-marker removal both happen before the spawn): held.
- D4, Drift rows switched to `decision-record` × D4's in-flight Drift-Baseline route: broken, low (F4).
- D4 × D10 (Record-Write sits after Preflight; Preflight stays the entry row): held.
- D4's goto, `step`, goto, `resume` route × the existing `lym loom` verbs: held, since `lym loom step` exists.
- D5 × D6 (the stop report leaves the record intact, so the discussion gate passes and the marker halts): held.
- D6 × D7, with no parent and a `Decision for <slug>:` note: broken (F1).
- D6 × D8 (the shared wrapper; the marker stays Darn-Write's): held.
- D7 × D4 (the record carries the body plus later decisions as the answer key): held.
- D9 × D4 and D7 (darn's order is unchanged; only tapestry's edges move): held.
- D12 × D8 (one builder; the plan perch budget comes from the tapestry module): held.
- D12 × D13 (`driver` stays in `loom.yaml`, `driver_kind` is retired; no overlap): held.
- D13's retired-key refusal × D13's own retirement and pruning, through `PrepareRunSettings`: broken (F2).
- D13 × the declared `PATTERN-shed-stop-chain` text: held.
- Acceptance criteria D1–D14 against their decisions: held, apart from the gaps F1–F4 name.
- Constraints against the decisions: held. `PATTERN-refusal-way-forward` is met in wording, but F3's loop leaves the gate-finding refusal without a working way forward.

**Findings**

**F1 — MEDIUM, design, D6 × D7 (decision-record.md, D6)**
Summary: D6 removes the writer's no-parent route that recorded a `Decision for <slug>:` note in the record, and puts nothing in its place.
- Today's stencil (`contracts/stencils/darn/darn-template-write.md`, Step 3, the no-parent bullet) has the writer re-read the board for the note, write its answer into the record as a Decision, and carry on.
- Under D6 the writer never edits the record, but it still reads `Decision for <slug>:` notes as an answer source (D6 line 120).
- With no parent, a resumed writer that finds an answering note has no stated action. If it follows the note, D7's perch measures against a record that lacks the answer and flags the deviation. If it stops again, the marker halts the run again, so a note-then-`resume` way forward loops.
- D6 names only `lym loom decision add` as the way forward. It never says that a note alone no longer settles a stop, or what the writer does with one.
Evidence: decision-record.md lines 120–125 (D6) and 133 (D7); darn-template-write.md, Step 3 no-parent bullet.

**F2 — MEDIUM, design, D13 (decision-record.md lines 196–199)**
Summary: D13 places its refusal "before `ValidateLocal`", but `PrepareRunSettings` runs `module.Migrate` before that point.
- `PrepareRunSettings` runs `moveRunSettings`, then `module.Migrate`, then `checkLocalDocument` (`internal/configreg/runsettings.go:46–84`).
- Loom's Migrate (`MigrateConfig`) drops every key in `retiredLoomKeys` (`internal/loomengine/config.go:709`, `:1057`).
- D13 has reconcile prune the retired keys. If that pruning goes through Migrate's drop list, as loom's other retired keys do, Migrate strips `loom.driver_kind` (or `arbiter*`, `tackler*`) from a profile or `run_settings` before D13's check sees it.
- The key is then dropped silently. That contradicts D13's own rationale (a run pinned to a driver must not silently run another) and its acceptance row (a profile naming `loom.driver_kind` is refused at write and at Seed-Child).
- The record states neither that the check runs before `module.Migrate`, nor that the pruning stays out of Migrate.
Evidence: runsettings.go:46–84; loomengine/config.go:660, 709, 1057.

**F3 — LOW, design, D4 × D5 × D6 (decision-record.md lines 88, 112, 121)**
Summary: a darn record the validate-only gate refuses has no working way forward.
- D5 keeps Darn-Write's `discussion` gate as a validate-only check that re-prompts the writer with the record's findings (attempts 3).
- D6 says the writer never edits the record.
- D4 says Record-Write never overwrites a file that exists, so a record torn by a crash mid-write, or one that fails the history check, is kept as it is.
- The re-prompted writer may not fix it, so the gate exhausts into Stuck carrying `darnResumeWayForward` ("lym loom resume", `internal/loomshed/darnwrite.go:15`, 93–95). That re-spawns the same failing gate, and `goto --to Record-Write` does not help because it never overwrites.
- The record states no way forward for a gate finding on the record itself on darn.
Evidence: darnwrite.go:15, 93–95; gates.go `ValidateDarnDiscussion`; D4 line 88; D5 line 112.

**F4 — LOW, consistency, D4 (decision-record.md lines 89, 95–96)**
Summary: a darn run parked on Drift-Baseline that is resumed without the goto spawns the baseline agent on a decision record that does not exist yet.
- D4 switches darn's Drift rows to `subject: decision-record`, and the agent is told to read the record at the decision-record path (`internal/driftengine/seats.go:62–63`). On a run parked before Record-Write, no record exists there.
- D4 bounds the "resumed without that goto" case only at Darn-Write. On the Drift-Baseline row the agent reads a missing file, and any Stuck it returns halts with drift's own ways forward, which do not name `goto --to Record-Write`.
- The run recovers later, when the Darn-Write refusal sends it back.
Evidence: driftengine/seats.go:60–64; decision-record.md lines 89, 95–96.

**Tool calls**
1. Tapestry and pawl recipe rows: an awk pass, because the first grep's output was garbled. I needed the row engines for D1 and D10; the manifest held only darn's recipe.
2. Board-entry snapshot callers (grep): to check whether D5 dropping the darn gate's commit breaks the pawl snapshot D7 keeps. The manifest lacked the writer's callers.
3. `lym loom --help`: to confirm the `step` and `goto` verbs that D4's in-flight route uses exist under `lym loom`.
4. Recipe engines plus the snapshot lines in darnwiring.go (an awk and grep pass): to get the engine of each row in the tapestry and pawl recipes, and to locate the snapshot writer.
5. darnwiring.go lines 170–196: what `commitDarnDiscussion` wraps.
6. Callers of `commitDarnDiscussion` and `writeBoardEntrySnapshot`: to confirm pawl's snapshot uses its own path (`pawlwiring.go:135`).
