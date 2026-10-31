```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens report on the decision record's bounds, guarantees and code claims.

**Findings**

1. **MEDIUM — D4, and D9's claim that resume by row name is safe.** A darn run already in flight when this lands can be left with no decision record.
   - Counterexample: a darn run is parked on Drift-Baseline, Drift-Await or Darn-Write, and its old writer has not yet written the record. It resumes by row name past the new Record-Write row, which sits before those rows.
   - The new writer stencil no longer writes the record. The writer row's validate-only `discussion` gate (D5) then fails on the missing record, and the run halts. `decision add` refuses the same run with "the run has no decision record" (`internal/loomcli/decision.go:373`).
   - The record says nothing about this. It needs a bound: either such runs are restarted, or the writer row's gate or the wiring writes the mechanical record when it is missing.

2. **MEDIUM — D9's claim that "a run in flight resumes by row name … continues along the new edges".** A run caught mid-review never runs Drift-Verify.
   - Counterexample: a tapestry run is inside the Code-Review perch under the old order (Parent-Align → Code-Review → Drift-Verify → Describe). It resumes after the upgrade, and the perch's exit now leads to Describe.
   - The record only covers the opposite case: a run parked on Drift-Verify after its review gets reviewed again. The skipped comparison is unstated, and no retired-row mapping or way forward covers it.

3. **MEDIUM — D11 and its acceptance criterion: "a seed recording `beam` is then refused as an unknown recipe, naming the available ones."** This is only true for the paths that write a seed or look a recipe up in the shed table.
   - `shedrun.ParseSeed` and `shedrun.ReadSeed` never call `ValidateRecipe`. Once the alias is gone, a `beam` seed decodes to `Recipe: "beam"`.
   - In `lym loom`, `resolveRecipe` then sees a recipe that is neither darn nor pawl. Today that falls through to tapestry routing and retired rows. Under D2 it hits the non-loom refusal, which names the `lym shed` verbs, not the available recipes.
   - `internal/reportengine/runs.go:252-257` reads seeds from git history through `ParseSeed` and skips a `beam` seed as a non-selected recipe. That path holds.
   - Fix: the claim and the criterion should name the entry point (`lym shed`'s lookup) or describe the actual refusal.

4. **MEDIUM — D13's claim that "a hub value tuned in `loom.yaml` or `darn.yaml` is dropped by the reconcile".** Run settings and profiles are a separate path that nothing rewrites.
   - `PrepareRunSettings` (`internal/configreg/runsettings.go`) says: "Nothing rewrites a profile or a run_settings document".
   - The record does not say whether `loom.arbiter`, `loom.tackler_max_spawns` or `darn.driver_kind` are refused or ignored when a task's `run_settings` or a `profiles.yaml` block carries them. If they are declared retired with a refusing category, every run using such a profile fails at start.
   - The Open risk names only hub `loom.yaml` and `darn.yaml` values. It should also cover profile and run_settings blocks and say what happens: refused or admitted.

5. **LOW — D3's claim that `lym loom peers` reads "each peer's row position against that peer's own recipe, read from the peer's seed".**
   - Counterexample: a peer owned by another machine, or not started yet, has no local worktree or seed, so its recipe cannot be read.
   - The board entry already carries a `recipe` field: `pawl-loom-recipe`'s `get` output shows `"recipe":"darn"`. That is a source that works for every peer. The record needs a fallback, or should use that field.

6. **LOW — D7 for darn runs already in flight.** A darn run past the old Darn-Write resumes at Code-Review with `board-entry.md` gone. Its answer key becomes a record the old writer wrote itself, which the darn recipe header calls "the writer's own claim, reviewed against the snapshot". The guarantee that the record holds the entry's body verbatim does not hold for these runs.

7. **LOW — D10's claim that Tapestry-Preflight's seed check "repeats the refusal Shed's own status read gives before every row".** The claim is taken from the board entry without checking it. `CheckSeed`'s doc says it checks a coherent, fresh seed for an expected producer, tolerates history entries, and has goto-reentry and half-finished waivers. Freshness is not obviously something Shed's status read refuses. The record should cite evidence or name what is lost.

**Claims attacked**

- D1, contract check via `on_done` path and terminals: held. The `on_reroute` edge from Finalize is outside `on_done`, and the darn path Record-Write → Drift rows → Darn-Write satisfies it.
- D2, pawl the only non-loom recipe `lym loom` serves: held, since batten goes through `battencli.ArmAt`. The `beam` remnant is covered in finding 3.
- D3, `--amends-plan` already refuses on darn: held (`decision.go:617`).
- D3, peers reading from the peer's seed: broken (finding 5).
- D4, Record-Write leaves an existing record untouched: held.
- D4, darn runs already in flight: broken (finding 1).
- D5, an empty section passes: held. The parser has no empty-section finding.
- D5, history check on tapestry: held.
- D7, run in flight: broken (finding 6).
- D9, resume by row name: broken (finding 2).
- D10, seed check redundant: not verified (finding 7).
- D11, `beam` refused as an unknown recipe: broken (finding 3).
- D12, `KeyMoves` carries tuned plan keys, run_settings included via `moveRunSettings`: held.
- D13, retired keys dropped: broken for profiles and run_settings (finding 4).

**Tool calls**

- Read `internal/reportengine` seed handling, to test D11 against history readers; my context did not have it.
- Read `internal/configreg/runsettings.go` and grepped runprofile, to test D12 and D13 on run_settings and profiles; not in context.
- Grepped `decision.go` for the missing-record refusal and read `CheckSeed`'s doc, to test D4's resume path and D10's redundancy claim; not in context.
