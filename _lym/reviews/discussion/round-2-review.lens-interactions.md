```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked every pair of decisions that share state, then each acceptance criterion and constraint against the decisions it touches.

**Pairs checked**
- D1 × D10 (record-only finish; end-as-Sketch skipped by row): held.
- D2 × D9 (sketch label only on a raddle task; filed entries carry no `sketch`, D6 refuses it): held.
- D2 × D3 (promotion keeps the flag and record; Record-Write seeds it): held.
- D3 × D9 (darn and pawl build from the stored part or project record): held. Darn's Record-Write row precedes Darn-Write, which reads the record as scope.
- D3 × D5/D9 tapestry shape (whole Sketch record; the Motif block's seed starts it at Plan): held.
- D5 × D6 × D8 × D9 retry (new slugs free; a retry after step 3 skips the re-check): held.
- D5/D6/D8 slug freshness × D10/D11/D14 raddle store keyed by slug: broken (F3).
- D6 carried criteria × D9 step 2 part records: broken (F2).
- D6 depends_on × D8 drop of a dependency on a gone entry: held.
- D6 auto gate attempts × D8 return cap: held, separate counters.
- D7 cold proposal × D9 step 1 staging × D8 deleting step-1 outputs: held, provided the names differ, which is plan detail.
- D7 approval hash × D8 return clearing approval × D9 hash match: held.
- D7 waiting notice × D8 return to a new interactive session: weak (F4).
- D7 recorded mode × D6 gate × D8 return "in the recorded mode": held.
- D8 baseline (last passing propose) × D7 raddle-approve × D9 filing re-check: held.
- D9 step 3 (parts released) × D9 step 4 (beam created): broken (F1).
- D9 step 3 (entry done) × D10 (no Sketch finish) × MakeSketch skipping done: held, consistent.
- D9 step 3 (entry done) × D9 retry of steps 4–5 after a halt: broken by the origin reconcile. That is existing-paths territory and the handler holds it, so it is not reported here.
- D9 step 5 notice × clearing notify_on_land in step 3: held.
- D10 store × D11 BeamUp lookup × step 4 ordering: held, since the store is written at step 3 before the beam exists.
- D12 Seed's stored-record flag × D3 × D13 cells: held.
- D12 per-session files × D13 sums × D14 actuals: held.
- D13 "no Finalize size record" × D1 raddle and motif: held.
- Every acceptance criterion against the decisions it names: consistent, apart from F1–F3.
- Constraints PATTERN-sole-parsers, PATTERN-shed-recipe-registry and PATTERN-shuttle-provider-seam × D3, D6, D7 and D12: held.

**Findings**

F1. BLOCKING, design. Location: D9 steps 3 and 4.
- Summary: for shape `beam`, step 3 files the parts released inside a plain project, and only step 4's `lym beam create` makes that project a beam.
- Consequence: from step 3's commit until step 4 completes, the parent level's dobby sees released tasks whose project carries no beam flag, so it can start them under the Sketch's parent instead of the beam. A halt between steps 3 and 4 leaves them that way indefinitely, which defeats the `beam` shape and D11's hand-off.
- Fix direction: file the beam shape's parts unreleased until step 4 has created the beam, then release them in a step with its own done marker, or set the beam flag in step 3's write.
- Evidence: `_lym/discussion/decision-record.md:132-139`. Beam membership comes from the project's `Beam` flag: `internal/boardengine/parentbeam.go:79-82` (`BeamUnder` falls back to `ParentBeamOf` while the project is no beam).

F2. MEDIUM, design. Location: D6 checks × D9 step 2.
- Summary: D6 requires each Acceptance criterion to be carried by at least one part, and each part's carried criteria to name every decision it builds. Nothing requires a carried criterion's ids to be built by that part or by a part it depends on.
- Consequence: a multi-id criterion can go to a part that builds only some of its decisions, and step 2 files that part with a criterion it cannot meet. The Sketch under review has such criteria, for example "D5 and D6: …".
- Fix direction: add a D6 check that every id a carried criterion or risk names is built by the part or by a part on its `depends_on` path.
- Evidence: `_lym/discussion/decision-record.md:71-72`, `:130`, `:224`.

F3. MEDIUM, design. Location: D5/D6/D8 slug freshness × D10, D11, D14.
- Summary: "every new slug is free" is judged against the board alone, which prune empties. The raddle store outlives prune, and it is looked up by the Sketch's slug or any filed slug (D10), by the beam's project slug (D11) and per proposal (D14).
- Consequence: a later Sketch, or a filed part, that reuses a pruned slug collides with an older store directory. Step 3 then overwrites it or the lookup becomes ambiguous, D11 mails a beam an old hand-off, and D14 loses or misattributes history.
- Fix direction: also check freshness against the raddle store, or key the store by something that cannot be reused.
- Evidence: `_lym/discussion/decision-record.md:65`, `:75`, `:108`, `:148-150`, `:155`, `:179`. Prune removes done entries: `internal/boardengine/store.go:790-796`.

F4. LOW, consistency. Location: D7 interactive × D8 return.
- Summary: after a return that does not move the board (a failing D6 check or a hash mismatch), the new interactive session needs a fresh `raddle-approve`.
- Consequence: the "proposal waits" notice goes out only on the first passing `raddle-propose` of a session, and nothing requires that call before `raddle-approve`. The parent orch may never learn that the run waits for the operator again.
- Fix direction: have `raddle-approve` refuse until the session has passed `raddle-propose`, or post the notice on the return.
- Evidence: `_lym/discussion/decision-record.md:92`, `:94`, `:111-112`.

**Tool calls**
- `ls contracts/recipes`, then grep `darn-recipe.yaml` rows: to confirm darn has a Record-Write row before Darn-Write (D3 × darn); the manifest lacked darn's rows.
- grep `contracts/stencils/darn` for the decision record: to confirm Darn-Write reads the record and does not rewrite it; not in the manifest.
- grep `internal/dobbyengine` and `BeamUnder`: to confirm a project's beam membership comes from its `Beam` flag (F1); not in the manifest.
