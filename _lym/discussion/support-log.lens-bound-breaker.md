```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked the record's claims, bounds and guarantees with concrete counterexamples.

**Findings**

1. **BLOCKING, D5: the sandbox root under `internal/lymtemp` does not outlive the verb that creates it.**
   - lymtemp's run directory belongs to one process. That process holds `run.lock` until it removes the directory, and `Root.Sweep` removes the run directories of dead owners.
   - `lym harden sandbox new` is a short-lived CLI process. A hub it builds in its own run directory is removed when the verb exits, or swept at the next lym start.
   - So `list` and `clear`, called from later processes, never see that root. The row-end reap, run in the shed driver's process, looks at a different root.
   - The "run's sandbox root" therefore needs a stable per-run location. lymtemp only provides per-process ones.

2. **MEDIUM, D7: the `harden` commit seam does not fire "on every settle".**
   - `shedadapters/bouncer.go` says the Commit seam "runs on the approved branch of settle, before Commit". `approveAndCommit` is called only on an approved settle.
   - Counterexample: a CONTINUE round, or a campaign capped at `max_bounces` or halted CIRCLING, archives nothing. That is exactly the evidence D7 means to keep, and the case where it matters most.
   - D7 needs a seam hook on non-approved settles, or the archive belongs to another row.

3. **MEDIUM, D2 against D9: `hardener.packages` and `hardener.llm_driving` are not config keys, but `run_settings` only carries config keys.**
   - `boardengine` doc: run_settings is "an object of module name to a mapping of that module's config keys".
   - D9's `hardener.yaml` key list omits `packages` and `llm_driving`. D2's charter values therefore cannot be written as run_settings without being `hardener.yaml` keys too, with defaults.
   - An empty default `packages` is what makes Preflight's refusal meaningful. The record needs to say this.

4. **MEDIUM, D5: the sandbox hub's `selfreport: false` and `friction: ""` are "written", not committed onto the prime records.**
   - The crucible template's selfreport hazard section says each slug's records branch forks from prime's committed records at that slug's create time. An uncommitted `loom.yaml` override does not reach pairs the round agent creates.
   - So a sandbox loom campaign can still run Tier 1 selfreport and Tier 2 friction against the real upstream. This is wider than the one Open risk the record names.

5. **MEDIUM, D1/D10: as a build row with a verify gate, `HardenFix` inherits darn's verify wiring.**
   - `loomrecipe/darnrecipe.go` `verifyGatedRows` gives every `LoomBuildEngines` row with a verify gate the tree_scope verify budget.
   - `loomcli/arm.go` takes that budget from "darn.yaml's for a recipe whose verify_source is tree_scope", and `darnwiring.go` `wireTreeScope` reads darn.yaml's `tree_scope` key.
   - Hardener's verify attempts and verify scope therefore come from `darn.yaml`. The record does not mention this coupling, and it conflicts with D9's claim that `hardener.yaml` holds the campaign's tuning.

6. **MEDIUM, D4: concurrent copies of `lym gate test --tier 2` compete for one pool of gate slots shared by the whole hub.**
   - `gateslot` bounds how many test runs the hub executes at once. A tier asking for `slots: all`, or more than the limit, is clamped to every slot, taken one at a time.
   - Counterexample: with `concurrency: 3` and a tier-2 tier that takes every slot, the copies run one after another, so the amplifier never runs concurrently. A smaller slot count instead holds the pool for the whole campaign and starves other tasks' gates.
   - The record states no bound for either case.

7. **MEDIUM, D5 against pause and crash: hubs from an interrupted row can outlive the run.**
   - The reap runs only "at the end of every … row". A run paused mid-row, or a shed driver that is killed, leaves sandbox hubs and their reed servers alive.
   - On resume, the re-run row's reap catches them only if the root path is stable across processes, which finding 1 breaks.
   - Nothing reaps at run end or at Finalize or teardown.

8. **LOW, D4: the deploy in step 1 does nothing for steps 2 and 3.**
   - `lym gate test` runs the tests against the gate's own prebuilt lym route (`LYM_PREBUILT_LYM`/`lymbin`), not `.dev-bin`.
   - Step 1 only refreshes the binary that the next round's live driving uses. The rationale "so every later step runs the committed code" is overstated.

9. **LOW, D4: the stray check `tmuxkit.Main` makes on processes reads `/proc` and runs only on Linux.**
   - D4 cites it without the platform limit that D5 states for its own scan.

10. **LOW, D8: the need for a hubreconcile reinterpretation of `agents.yaml` is not established.**
    - A grep of `internal/agentconfig` found no refusal of an unknown role.
    - If the loader tolerates a removed role, the reinterpretation is unnecessary work. If it refuses one, the claim holds. The record should say which.

**Claims attacked**
- D1, darn shape, dobby start and Finalize landing: held. The board's `recipe` field and `RecipeOf` exist.
- D2, Preflight refusal from run_settings: broken (finding 3).
- D3, burler review's single write surface: held. `.dev-bin/` is gitignored, so the reviewer's `deploy-dev` leaves no dirt.
- D3, a recipe cannot configure the live gate off: held as designed.
- D4, serial gate and concurrent copies as diagnostics only: broken by slot contention (finding 6). Deploy claim overstated (finding 8). Stray-check platform limit unstated (finding 9).
- D5, sandbox root lifetime: broken (finding 1). Config override: broken (finding 4). Reap coverage: broken (finding 7). The reap's socket-key bound holds, since reed uses `-L`.
- D6, Bouncer facts and existing escalation: held as a design. The facts need a seam that `bouncerfacts.go` lacks today, which is plan-level work.
- D7, per-settle archive: broken (finding 2).
- D8, treadle has no production importer: held. The reconcile claim is unverified (finding 10).
- D9, `hardener.yaml` holds the tuning: broken by darn.yaml coupling (finding 5) and the missing keys (finding 3).
- D10, recipes without a segmented build row are unchanged: held. `verifyGatedRows` picks up `HardenFix` (finding 5).
- D11, cobra wiring: held.
- Open risk, a hand-run `lym selfreport create` stays possible: held, and wider than stated (finding 4).

**Tool calls**
1. Read the lymtemp package doc, the `.dev-bin` line of `.gitignore` and the build-row consumers in loomrecipe and loomcli: lymtemp lifetime and darn coupling were not in my context.
2. Read `darnrecipe.go` `verifyGatedRows`, the Bouncer commit-seam trigger in `bouncer.go`, the boardengine run_settings doc and `RecipeOf`: commit timing and run_settings shape were not in my context.
3. Read the gateslot doc and grepped agentconfig for an unknown-role refusal: slot-pool semantics and role tolerance were not in my context.
