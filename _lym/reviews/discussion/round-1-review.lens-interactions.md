```yaml
lens: interactions
outcome: reported
```

Interactions lens: I checked pairs of decisions that act on the same state, and checked each acceptance criterion and constraint against the decisions it touches.

**Pairs checked**
- D1 × D2 (whether `contract: loom` decides what `lym loom` serves, plus the pawl exception): held
- D1 × D4 (record-row and writer-row engine sets, with Record-Write before Darn-Write on the `on_done` path): held
- D1 × D8 (D1 requires `verify_source`; pawl has no contract and keeps its source through D2): held
- D1 × D9 (Finalize's `on_done` successor is a terminal; `on_reroute` is not under the clause): held
- D1 × D10 (the Implementer engine set and Preflight's row-keyed `batcher.yaml` load): held
- D2 × D11 (`beam` is refused as non-loom, and the exception names pawl alone): held
- D2 × D13 (D2 says pawl keeps every path; D13 changes pawl's override source): **broken** (F3)
- D3 × D4 (`decision add` before Record-Write; `--amends-plan` keyed on Plan-Write): held
- D3 × D5 (`--pattern-*` bullets on darn kept by the shared history check): held
- D3 × D8 (`peers` per-recipe rows against the one builder per name): held
- D4 × D5 (D4's record passes the one validator; citation and relevant checks skipped by record-row engine; extra H2s from a board body are no finding, per `internal/discussionparser/validate.go:62-64`): held
- D4 in-flight halt × D4 row placement before Drift-Baseline: **broken** (F1)
- D4 × D6 (the writer never edits the record; Record-Write's commit goes to the weft repo, so "no commit past the fork" still counts only code commits): held
- D4 × D7 (darn perch reads the D4 record): held
- D6 × D7 (no-parent outdated-entry route against the record as the sole answer key): **broken** (F2)
- D6 × D8 (shared writer wrapper; the no-commit halt belongs to Darn-Write, a darn-only producer): held
- D7 × D2 (snapshot writer kept for pawl): held
- D8 × D13 (`RecipeOverride`'s recipe-name branch goes with D13): held
- D9 × D1 / Finalize reroute: held
- D10 × D8 (retired-row data per recipe; Tapestry-Preflight → Discussion-Write): held
- D12 × D13, profile and run_settings read path (KeyMoves translation and the retired-key refusal share `PrepareRunSettings`): **broken** (F4)
- D12 × D13, reconcile (`KeyCopies` loom→darn `arbiter*` against keys D13 retires in both modules): **broken** (F5)
- Acceptance criteria D1–D14 against their decisions: held. D8's "no recipe-name branch" against D2's unseeded-reads-as-tapestry default reads as a default, not a branch.
- Constraints (registry, sole parsers, refusal way-forward, gate parity, config layers, stencil ownership, tolerant records, retired rows, Preflight entry): held

**Findings**

**F1 — MEDIUM, design, D4 (in-flight darn run with no record)**
The halt leads to a way forward that fails. The `discussion` gate only runs at the writer's handoff (`contracts/recipes/darn-recipe.yaml`, Darn-Write `gates: discussion attempts 3`). By then the writer session has already committed code on the task branch. Following `goto --to Record-Write` then routes through Drift-Baseline, whose precondition needs HEAD to equal the branch point (`internal/driftengine/doc.go:29`), so the named recovery is refused whenever drift is enabled. The reason D4 claims is also not what the code produces. An exhausted gate's Stuck reason is the validator's findings plus `darnResumeWayForward`, a resume (`internal/loomshed/darnwrite.go:93-95`), and a resume re-runs Darn-Write into the same halt. D4 needs to say how the reason comes to name `goto --to Record-Write`, and what the run does about the code already committed when it routes back through Drift-Baseline.

**F2 — MEDIUM, design, D6 × D7 (no-parent outdated-entry stop)**
D6 forbids the writer to edit the record, and with no parent nobody can run `lym loom decision add`; the verb refuses a task session (`contracts/stencils/darn/darn-template-write.md`, scope-correction section). So the answer to a `Decision for <slug>:` note reaches the resumed writer only as an unrecorded correction. D7 makes the record the sole answer key and drops the board-entry snapshot. The Code-Review perch therefore judges a change that follows the note against a record without it. Today's stencil had the writer write the note's answer into the record (no-parent bullet, same file). D6 needs to say how a no-parent answer enters the record, or how review accepts it.

**F3 — MEDIUM, consistency, D13 vs Scope Out / D2**
D13 moves pawl's arbiter, tackler and driver-kind source from `loom.yaml` to `shed.yaml` ("every recipe, pawl included"). It also refuses a profile's pawl block that names `loom.driver_kind`. Today pawl reads `loom.yaml` through `RecipeOverride`'s non-darn branch (`internal/loomengine/config.go:1151-1166`). That contradicts Scope Out ("every pawl path stays as it is") and D2 ("every wiring path it uses today"). One of them must name the exception.

**F4 — LOW, consistency, D13 refusal point**
D13 says the retired-key refusal fires "when the run starts". It shares `configreg.PrepareRunSettings` with D12's on-read translation, and that function runs at board write and Seed-Child (`internal/boardcli/runsettings.go:90`, `internal/battenshed/seamchild.go:150`). Per `internal/runprofile/doc.go` it also runs on every write of `profiles.yaml`. So the refusal also hits `lym board` run_settings writes and profile writes, the path the orch uses for the rewrite D12 and Open risks rely on. D13 should state the real refusal points.

**F5 — LOW, design, D13 × `KeyCopies`**
`configreg.KeyCopies` copies `loom.arbiter*` into `darn.arbiter*` in hub base files (`internal/configreg/keymove.go:51-57`, applied by `internal/configsync/movekeys.go:252`). D13 retires those keys in both modules and says the reconcile drops a hub value, but is silent on the copy entries. Depending on order, a reconcile copies a retired key into `darn.yaml` before or after dropping it, and the entries stay as dead declarations over retired keys. D13 should remove them.

**Tool calls**
1. Grepped `internal/discussionparser` and the configsync/hubreconcile files for section findings and the `KeyCopies` application: the manifest held neither the record parser's section rules nor the reconcile's copy path.
2. Read `internal/discussionparser/validate.go:20-60` and `internal/configsync/movekeys.go:222-262`: to learn whether a board body's extra H2s fail the D4 record, and how `copyKeys` treats a source key.
3. Read `internal/discussionparser/validate.go` heading/order lines: to confirm extra H2s and section order are not validated.
