```yaml
lens: interactions
outcome: reported
```

Interactions lens: I checked every pair of decisions that act on the same state, key or ordering, then each acceptance criterion against the decisions it touches.

**Findings**

1. **BLOCKING — D3 × D6 × D9: under the record's own defaults the campaign can no longer converge after round 3.**
   - D3 and its acceptance criterion pick round n's model as "the nth model … clamped to the last". The exploration confirmed burler's `RoundModels` clamps the same way.
   - D9's default list is Opus, Fable, Sonnet, so every round from 3 onward gets Sonnet.
   - D6 requires CONVERGED to have "a reviewer model different from the previous round's". That can never hold for rounds 4 to 7 under `max_bounces: 7`.
   - Result: a campaign that is still open after round 3 always ends at the cap and escalates.
   - Fix: rotate cyclically, which means a burler change, or make the rule "differs from the model of the round that last closed a finding".

2. **BLOCKING — D5 × the PATTERN-lym-files constraint: the "run's sandbox root" has no owner that outlives its creator.**
   - The `lymtemp` doc says each process owns one locked run directory, holds `run.lock` until it removes it, and `Root.Sweep` removes the directories of dead owners.
   - `lym harden sandbox new` runs as its own short-lived CLI process from the agent's pane. When it exits, its directory (and the hubs in it) is either removed or swept while the reviewer is still driving them.
   - The row-end reaper runs in the shed driver, a different process with a different run directory, so it cannot compute the same root.
   - A shed resume after a crash is another new process, and it would orphan the old root as well.
   - D5 has to name a stable, run-keyed root: under `lymtemp`'s root but outside any one process's run directory, or under the run's `.lym` path. It also has to make the verbs and the reaper derive that root the same way.

3. **MEDIUM — D7 × the Bouncer settle semantics: the archive happens only on approval, not "every settle".**
   - `shedadapters/bouncer.go`: Commit "runs on the approved branch of settle"; approveAndCommit is called only on the approve and accept paths.
   - So the `harden` seam fires once, at convergence or on a parent's accept. Rounds of a capped or circling campaign that is not accepted are never archived.
   - D7's purpose, evidence that survives teardown, therefore fails exactly in the campaigns that need it most.
   - D7 needs its own per-round hook, or should say it archives only on settle-approved.

4. **MEDIUM — D1 (`verify_source: tree_scope`) × D3 (`verify` gate) × D9: the verify scope and attempts come from `darn.yaml`.**
   - `loomcli/darnwiring.go`: `wireTreeScope` fills the verify Env from `darn.yaml`'s `tree_scope`.
   - `loomrecipe`: a tree_scope recipe "takes env.DarnVerifyAttempts".
   - So HardenFix's `verify` gate, and Finalize's landing gate, are scoped by darn's config, while D2 and D4 scope the campaign by `hardener.packages`. The record says nothing about this coupling.

5. **MEDIUM — D2 × D9: `hardener.packages` and `hardener.llm_driving` are not in the `hardener.yaml` key list.**
   - D2 puts both in `run_settings` under the `hardener.` namespace. D9 says `run_settings` may override any `hardener.yaml` key but lists neither of them.
   - If `run_settings` keys are checked against the unit's keys through configengine, D2's settings are refused. If they are not, D9's "override any key" is not the mechanism D2 relies on.
   - The record needs to settle one model: either both keys are unit keys with an empty default, or they are task-only settings with their own reader.

6. **MEDIUM — D4 × D5: the first round drives a stale or missing binary.**
   - D5 makes the `.dev-bin` lym the only binary a round uses, but the only deploy in the design is the live gate's (D4), which runs after the round 1 fix.
   - Round 1's reviewer therefore drives whatever `.dev-bin` already holds, or nothing.
   - The fixer's mid-round re-drives also run undeployed code. The rubric rules listed in the acceptance criteria omit "deploy-first after every change" from crucible.
   - Fix: deploy in Preflight or before each review row, and add deploy-first to the rubric's acceptance criterion.

7. **MEDIUM — D9 × the Bouncer and loomrecipe sources: the round cap and model keys have no wiring.**
   - `loomrecipe` reads the perch budget only from `loom.yaml` (`<perch>_max_bounces` or `review_max_bounces`), and the Bouncer row refuses model keys, taking its judge and seed models only from `loom.yaml`.
   - D9's `max_bounces`, `judge`, `seed` and fan keys in `hardener.yaml` therefore never reach the Harden-Review rows unless loomrecipe gains a hardener source, which no decision states.
   - As written, the round cap silently falls back to `review_max_bounces`.

8. **LOW — D5 row-end reap × D6 stray attribution.** After a crash or pause the row-end reap never runs. The next row's reap then records the previous round's leftovers as this round's stray state. This depends on finding 2's fix.

9. **LOW — D4 stray check × the Constraints claim.** `tmuxkit.Main`'s stray-process failure covers only packages whose TestMain calls it. A `hardener.packages` entry without it gets no stray check, and the record does not bound that.

10. **LOW — D3: the hub gate key HardenFix's `verify` gate runs is never named.** Code-Burler-Fix runs `code-burler-fix` through `gate_key`; HardenFix's key is left unstated.

**Pairs checked**
- D1 × D2 (Preflight check of the charter settings): held.
- D1 × D10 (contract and ring placement): held.
- D1 × D4 (verify_source vs live gate scope): broken, finding 4.
- D2 × D9 (`run_settings` keys): broken, finding 5.
- D3 × D6 × D9 (model rotation vs convergence rule): broken, finding 1.
- D3 × D10 (HardenFix as build row): held.
- D3 × D4 (gates fixed on HardenFix): held. Verify gate key unnamed, finding 10.
- D4 × D5 (deploy vs `.dev-bin` use): broken, finding 6.
- D4 × D5 (live gate's tmuxkit TMUX_TMPDIR vs sandbox reap): held.
- D5 × lymtemp / PATTERN-lym-files (root ownership): broken, finding 2.
- D5 × D6 (stray report into facts, Bouncer after fix row): held. Restart path broken, finding 8.
- D5 × D11 (verbs refuse outside a hardener worktree): held.
- D6 × D7 (facts vs archive): held.
- D7 × the Bouncer commit seam semantics: broken, finding 3.
- D6 cap and circling × the done-when "unattended to convergence or cap": held, since the halt goes to the parent.
- D8 × D12 (treadle and Tenter removal vs doc updates): held.
- D8 × PATTERN-config-layers acceptance criterion: held.
- D9 × loomrecipe budget and model sources: broken, finding 7.
- D10 × other recipes (only segmented build rows read): held.
- D11 × PATTERN-cli-cobra: held.
- D12 × the rubric acceptance criterion: deploy-first missing, finding 6.
- D5 / Open risks × PATTERN-github-auth constraint: consistent.

**Tool calls**
- Read the `internal/lymtemp` package doc: the context did not cover how run directories are owned and swept, which D5's root rests on.
- Grepped the Bouncer's commit seam call sites in `internal/shedadapters/bouncer.go`: the context did not cover when Commit fires, which D7 rests on.
- Grepped how `internal/loomrecipe` sources the budget and judge settings: the context lacked how D9's keys would reach the rows.
