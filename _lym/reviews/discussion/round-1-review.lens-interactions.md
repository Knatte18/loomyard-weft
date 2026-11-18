```yaml
lens: interactions
outcome: reported
```

Interactions lens: I checked the record's decisions against each other. Five pairs are broken (findings 1 to 5). Two more findings come from the same pairs: one involves D9's run_settings override and one is the criteria gap of finding 1.

**Pairs checked**
- D3 cyclic model lists × D6 CONVERGED "different reviewer model" × D9 `max_bounces`: broken (finding 1)
- D9 `run_settings` override of `reviewer` × D6 CONVERGED rule: broken (finding 2)
- D4 live gate "in held gate slots" × D4 `--copies` "inside the slots the call holds": broken (finding 3)
- D5 reaps (four per round, two rows) × D6 one per-round stray report × D7 archive at fix-row end: broken (finding 4)
- D4 live gate re-prompts the fixer × D6 fixer defers out-of-module findings × D6 CONVERGED needs a green live gate: broken (finding 5)
- D3 HardenFix `verify` runs `harden-burler-fix` × D1/D9 `tree_scope` as the run's verify scope: broken (finding 6)
- D3 fixer commits per fix × D4 live gate on the committed tree: held
- D3 fixer commits (code side) × D7 archive commits through fabric (records side): held
- D6 carry-over into `## Open risks` × D3 the decision record as fasit: held, and the next reviewer sees the deferrals by design
- D7 per-round ledger copy × D7 final-ledger seam on an approved settle: held. A round-n archive holds the ledger the Bouncer wrote before round n, and the final one comes from the seam.
- D10 HardenFix in `LoomBuildEngines` × D1 recipe rows × the one-build-row check (`onlyRowOf`): held. It covers the two-HardenFix criterion.
- D10 × `recordOnly`: held. A recipe holding HardenFix is no longer record-only, as intended.
- D2 Preflight `packages` check × D9 defaults and override × D11 hardenshed: held
- D1 arm reads `hardener.yaml` × D9 keys: held, apart from finding 6
- D5 sandbox root under `.lym` × D7 archive under `_lym` × the PATTERN-lym-files Constraint: held
- D5 sandbox reaps × D4 tmuxkit stray check: held. They are disjoint: tmuxkit uses a private `TMUX_TMPDIR`, and the reaps use the root's socket keys.
- D5 verb refusal outside a hardener worktree × D5 Go-side reaps: held. The reaps never go through the verbs.
- D8 treadle removal × D3/D6 Bouncer rubric stencil: held
- D8 × D12 doc cleanup × criterion "no doc names Tenter": held. The grep finds Tenter only in the glossary and overview, which D12 names.
- D3 fans × D9 `reviewer_fan`/`fixer_fan` (not expanded cyclically, so they clamp): held. Convergence does not depend on fans.

**Findings**

1. MEDIUM · design · decision-record.md D3/D6/D9
   - D3 expands the reviewer and fixer lists only to `max_bounces`. A parent "continue" after the capped escalation runs rounds past that cap, up to the Bouncer's HardCap. Those rounds clamp to the last expanded entry.
   - So round cap+1 runs the same reviewer as round cap: with a cap of 7 and three models, rounds 7 and 8 both get Opus. D6's different-model CONVERGED rule then cannot hold in any continued round.
   - D9 states no hard cap for the Harden perch. The criterion at line 218 covers only rounds 1 to 7.
   - Evidence: internal/burlerengine/models.go:18-31 (clamp to the last entry); internal/shedadapters/bouncer.go:144-147 and 1253-1255 (continue below HardCap, HardCap separate from the budget); internal/loomcli/wiring.go:1427 (soft caps).
2. MEDIUM · design · decision-record.md D9 (line 158) × D6 (line 120)
   - "A task's run_settings may override any of them" lets a campaign set a one-entry `reviewer` list, or a list with equal neighbours.
   - Every round then runs the same reviewer as the round before, so CONVERGED can never be ruled and the campaign always runs to the cap.
   - The override has no stated bound, such as refusing lists that are too short or that have equal neighbours.
   - Evidence: models.go:28-31 clamp.
3. MEDIUM · design · decision-record.md D4 (lines 72, 80-81)
   - The live gate runs "in held gate slots" and then calls `lym gate test --tier 2 --copies <n>`.
   - A `lym gate test` started inside a slot an enclosing gate run holds acquires no slot of its own. So `--copies` gets only the enclosing gate's slots, and a non-tier holder holds one.
   - "One copy per held slot" then runs a single copy every round. The concurrency amplifier never runs, and the live report records the shortfall every time.
   - Evidence: internal/gatecli/cli.go:81; internal/gatecli/run.go:150-167 (`enclosingGateEnv` is inherited); PATTERN-gate-slot background (a non-tier holder holds one slot).
4. MEDIUM · consistency · decision-record.md D5 (line 98) × D6 (line 115) × D7 (line 130)
   - D5 writes "the round's stray report" from four reaps per round: start and end of the review row, then start and end of the fix row.
   - D6 lists one stray-report file, and D7 copies it once, at the end of the fix row.
   - Neither decision says whether those writes append or go to separate files per row. So the review row's end-of-row record, the strays the reviewer itself left, can be overwritten by the fix row's reaps before the archive copy and before the facts read it.
   - Criterion line 224 does not settle this.
5. MEDIUM · design · decision-record.md D4 (line 72) × D6 (lines 120, 122)
   - The `live` gate re-prompts the fixer "like any row gate". D6 has the fixer defer any finding whose fix lies outside the module.
   - A tier-2 failure whose cause the fixer defers keeps the gate red: the re-prompts exhaust the gate's attempts. No decision says what follows. The row may halt the run, which makes deferral impossible for live-gate failures. Or the round may end red, and CONVERGED then waits on a fix the rubric forbids.
   - D9 also sets `verify_attempts` but no attempt count for the live gate.
6. LOW · consistency · decision-record.md D3 (line 61) × D1 (line 38)/D9 (line 156)
   - D3 has HardenFix's `verify` gate run the hub key `harden-burler-fix`.
   - D1/D9 make `tree_scope` "the run's verify scope" and `verify_attempts` "the verify gate's attempts".
   - In darn, the verify gate runs "the tree-scoped command, or the hub's code-burler-fix key". The record leaves open which of the two HardenFix's verify runs.
   - Evidence: contracts/recipes/darn-recipe.yaml:235; internal/loomcli/arm.go:351-361.
7. LOW · consistency · decision-record.md Acceptance criteria line 218
   - The model-rotation criterion pins only rounds 1 to the cap. It leaves untested the continued rounds of finding 1, where the clamp breaks D6's rule.

**Tool calls**
- Grepped `enclosing` in gatecli/gateslot and HardCap/continue in `shedadapters/bouncer.go`. My context held neither the nested-slot rule's implementation nor the Bouncer's continue-past-budget path.
