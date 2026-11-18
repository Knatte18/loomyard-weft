```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end against the code it relies on.

**Findings**

1. **BLOCKING, D3/D6/D9: model rotation stops at round 3, so the convergence rule can never be met after that.**
   - D9 makes the model lists `[Opus, Fable, Sonnet]`, and the round cap defaults to 7.
   - Burler's `RoundModels` picks the model by round number and clamps to the last entry. So rounds 3 to 7 all run Sonnet.
   - D6's CONVERGED rule needs "a reviewer model different from the previous round's". From round 4 on that never holds, so a clean campaign runs to the cap and halts as capped instead of converging.
   - Evidence: the burler agent's report ("picked by round number and clamped to the last entry") and `internal/shedrecipe/recipe.go` holding `Models burlerengine.RoundModels`.
   - Fix: make the rotation cycle (modulo), or drop the different-model clause.

2. **MEDIUM, D7: the commit seam fires only when the Bouncer approves, not "on every settle".**
   - `approveAndCommit` is called only on verdictConverged, on circling-accept and on the replay paths. CONTINUE rounds never reach `cfg.Commit`.
   - So the per-round archive under `_lym/harden/round-<n>/` is written only at the final settle.
   - A capped or circling halt the parent never accepts archives nothing, which is the crash and teardown case D7 exists for.
   - Evidence: `internal/shedadapters/bouncer.go` lines ~539, ~779, ~1215, ~1338, and the settle doc at ~1164.
   - Fix: D7 needs a hook on unconverged settles too, or must say that it archives all rounds at once on the terminal settle and accept the loss on halt.

3. **MEDIUM, D2/D4/D9: three settings are read through `run_settings` but are not keys of the `hardener.yaml` unit.**
   - D2 and D4 read `hardener.packages`, `hardener.llm_driving` and `hardener.concurrency`.
   - D9's key list holds `concurrency` but not `packages` or `llm_driving`.
   - If `run_settings` are unit keys validated through configengine, Preflight's read of `hardener.packages` has no key to read, or the write is refused as unknown.
   - Fix: list `packages` (no default, empty) and `llm_driving` (default false) in D9.

4. **MEDIUM, D3: HardenFix's `verify` gate names no gate key.**
   - The verify gate runs the row's `gate_key`, otherwise the row name in lower case. Here that is `harden-burler-fix`.
   - The record adds no such key to the hub's `gate.yaml` and sets no `gate_key`, so the first fix round's verify gate has no key to run.
   - Evidence: `internal/shedrecipe/entries_gate.go:77`, and Code-Burler-Fix running the hub's `code-burler-fix` key.

5. **MEDIUM, D1/D10: with `verify_source: tree_scope`, darn's settings drive the hardener run's verify.**
   - `loomCLI.verifySource` returns `darnVerifySource` for any tree_scope recipe.
   - `loomrecipe.verifyGatedRows` hands every build row that has a verify gate darn.yaml's `verify_attempts`. HardenFix becomes such a build row under D10.
   - So darn.yaml's `tree_scope` and `verify_attempts` set the verify scope of a hardener run, its fix-row attempt budget and Finalize's landing verify. The record does not say this.
   - Evidence: `internal/loomcli/arm.go` around 350 and `internal/loomrecipe/darnrecipe.go` `verifyGatedRows`.

6. **MEDIUM, D6/D5: the facts file reads only review files and ledgers today.**
   - `computeRoundFacts` reads rounds 1..n review files and the earlier ledgers in runDir.
   - The models, live report, stray report and fix commits D6 adds have no producer that writes them where the facts renderer reads.
   - D6 states the outcome but names no writer for the per-round models record or the fix-commit list. The live and stray reports are written by the fix row and the row-end reap, but where they live in the run dir is unstated.
   - This is a seam the plan writer must invent. A design-level statement would be enough: "the HardenFix row writes `round-<n>-live.md` and the row-end reap writes `round-<n>-stray.md` into the shared run_subdir".
   - Evidence: `internal/shedadapters/bouncerfacts.go:147-151`.

7. **LOW, D5: the reaper keys on reed socket names derived by the running lym.**
   - When the module under test is reed and a round changes how socket keys are derived, the `.dev-bin` binary's servers carry keys the reaper does not compute.
   - The `/proc` scan catches pane processes by cwd, but the tmux server's own argv carries only `-L <key>`. On Linux the server can survive the reap and go unrecorded.

8. **LOW, D4 step 1: the deploy does not serve the gate's own steps.**
   - `lym gate test` compiles from source, so the deploy feeds only later agent driving, not steps 2 and 3. The record says it makes "every later step run the committed code".
   - No harm, but the rationale is wrong.

**Per decision: the case I used, and whether the purpose is met**

- D1 (the recipe): darn-shaped walk Preflight → Record-Write → Harden-Bouncer (entered first with zero rounds, as Code-Bouncer is) → Describe → PR-Review → Finalize. Met, apart from finding 5.
- D2 (the charter): an empty `hardener.packages` should be refused by Preflight. Not met while the key is missing from the unit (finding 3).
- D3 (the round): a fix round's verify gate. Not met (finding 4). The rotation is broken (finding 1).
- D4 (live gate): a red serial run re-prompts the fixer, a green one passes, concurrent failures only reach the report. Met; the rationale needs fixing (finding 8).
- D5 (sandbox): a reviewer makes two hubs and leaves one reed server alive; the row-end reap records and kills it. Met, except the changed-key case (finding 7).
- D6 (facts and convergence): round 4 is clean. Not met (findings 1 and 6).
- D7 (archive): round 2 settles CONTINUE. Not archived (finding 2).
- D8 (retire treadle): remove the package, stencils and roles, then reconcile. Met.
- D9 (config unit): incomplete key list (finding 3).
- D10 (contract): HardenFix is placed at Harden-Bouncer, which lies after Record-Write. Met.
- D11 (CLI): cobra subtree wiring. Met.
- D12 (crucible): deletion and docs. Met.

**Tool calls**

1. Grepped `bouncerfacts.go`, the gate-name registry and the RoundModels wiring: I needed the facts inputs and the gate-name source, which my context lacked.
2. Read `darnrecipe.go` and `loomcli/arm.go`: to see how a tree_scope recipe gets its verify settings, which my context had only as grep lines.
3. Grepped the Bouncer's Commit call sites: to find when the commit seam fires, which my context lacked.
4. Read the contexts of those call sites: to confirm commit runs only on the converged and accept paths.
