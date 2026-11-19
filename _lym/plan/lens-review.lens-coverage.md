```yaml
lens: coverage
outcome: reported
```

Coverage lens: I checked every decision, acceptance criterion and Intent branch of the Hardener plan for a planned test row that would fail if it broke.

1. **MEDIUM.** D1 criterion "a board task with `recipe: hardener` is started by Run-Shed through dobby and walks Preflight, Record-Write and the Harden-Review ring" has no row that drives the walk.
   - Card 13 plans only `TestChildRecipeSupported` and the `TestHardenerRecipe` parse and routing row.
   - Card 14's `TestWire_HardenerRecipe` checks wiring fields only.
   - No row builds and steps the recipe through its rows.

2. **MEDIUM.** D3 criterion "the Harden-Burler-Review row spawns with tool use, source fix scope and the `harden-rubric-review` rubric" is unreached. Card 13 sets these in YAML, but no row asserts the built review row's profile. `TestHardenerRecipe` only parses, checks the contract and projects routing.

3. **MEDIUM.** D3 criterion "the `verify` gate runs the `harden-burler-fix` key, over the touched trees' targets when the round diff lies wholly under `tree_scope`" is reached by no row.
   - Card 11's `TestHardenFixEntry` asserts the gate order and attempts, not the gate key.
   - Card 14's `TestWire_HardenerRecipe` asserts only the negative: a `darn.yaml` edit leaves the scope unchanged. No row shows `hardener.yaml`'s `tree_scope` narrowing the `harden-burler-fix` call.

4. **MEDIUM.** D1/D9 criterion "a `hardener` run's verify attempts come from `hardener.yaml`, and changing `darn.yaml` leaves them unchanged" has no row for the attempts half. Card 14's row covers only the tree scope. Card 11 takes `verify_attempts` from `Env.Harden`, but no row changes `darn.yaml`'s `verify_attempts` and checks the HardenFix verify entry.

5. **MEDIUM.** D4/D6 criterion "when `live_attempts` run out, the row ends red with the failures deferred, the run goes on, the next Harden-Bouncer's facts carry both, and the judge does not rule CONVERGED" has no row past the flag.
   - Card 11 only checks that `PassOnCap` is set on the `live` entry.
   - `internal/shuttleengine/gate.go:106` confirms that `PassOnCap` lets the run through after `Attempts` failures, but no planned row drives `live_attempts` failures through the HardenFix row.
   - No row shows the next round's facts carrying the red mark and the deferred findings.

6. **MEDIUM.** D4 criterion "runs both through `lym gate test --gate` with `packages` as targets and names no tier", and the rule that each live call is top-level and takes its own slots, have no row.
   - Card 6's `TestLiveGate` fakes `Run`. The real command (`--wait --gate <key> [--copies n] <packages>`) is built in card 14's `hardenCampaign` and asserted nowhere.
   - Neither card says the spawned call's environment drops `gateslot.InheritEnv` (`LYM_GATE_SLOT`). A child that inherits it would run inside a held slot, the nesting D4 forbids.

7. **MEDIUM.** Card 11's branches have no rows:
   - `bouncerEntry` with `harden_round: true` refusing a nil `Env.Harden`;
   - `bouncerEntry` filling `RoundFacts`, `LiveRed` and `OnHalt`;
   - `burlerReviewEntry` with `harden_round` refusing a nil `Env.Harden` and wrapping its producer with `RowReview`.

   Only the `harden` commit seam row and the admission in `TestBurlerHalfEntries_StrictUnknownKeys` are planned. A missing fill would ship a Harden-Bouncer with no facts, no red bound and no halt reap.

8. **MEDIUM.** D3/D4 says lym's own hub sets `harden-burler-fix` to `[0, 1]` and both live keys to `[2]`; card 21 does not carry this reliably.
   - Its `**Custom:**` target is `_lym/config/gate.yaml`. The task worktree has no `_lym/config/gate.yaml`; the hub's file is `_bolt/_lym/config/gate.yaml` (read: only `beam.yaml`, `loom.yaml` and `sparse` exist under the worktree's `_lym/config`).
   - Its Intent makes the outcome conditional ("when the default-layer write refuses … changes no file"), which is a research step the rubric flags.
   - The hub write is outside the worktree, so it is a parent call that should be recorded rather than left to the implementer's discovery.

9. **MEDIUM.** Card 1's lease and copy-count arithmetic does not cover a tier whose slot count is 0 (`slots: all`, per `gateslot/tiers.go:277-281`).
   - "n times the tier's own slot count" gives 0, and "held slots divided by the tier's slot count" divides by zero.
   - Neither the Intent nor a row covers this branch.
   - lym's hub tier2 sets no `slots` (default 1; `_bolt/_lym/config/gate.yaml:24-31`), but the flag is generic.

10. **LOW.** Card 9's round number for the review row's end-of-row reap is ambiguous.
    - The Intent sets the round as `ReviewRound` plus one for a review row. Once the review row has written review n, `ReviewRound` returns n.
    - If step 5 recomputes the round, the review row's end-reap strays go to round n+1's stray report and miss round n's archive.
    - That breaks the criterion "a stray left by the review row appears in round-<n>'s archived stray report", and `TestWrapRow` does not pin which round the review row's end reap writes to.

11. **LOW.** D2 names "the hub's own `gate.yaml`", but card 2's `HubSetsGate` also counts a worktree or machine layer. That is a near operand: a task worktree's own `gate.yaml` would pass Preflight's hub-key check.

12. **LOW.** Card 5's tier check passes only the key's tiers to `HoldsTierTest`. `lym gate test` narrows with the tier plus `tier.Covers` (`tierTestTargets`), so a package covered only through `covers` would be refused although the live run starts its binary.

13. **LOW.** These branches and exits have no row:
    - card 8: a `StopServer` error keeping the list entry, and a reap error;
    - card 9: a reap or file error joined to the wrapped call's error;
    - card 10: a `LiveRed` error escalating;
    - card 16: `new` and `list` in a worktree holding a root but no hardener seed, a worktree with no seed, and `new`'s printed JSON with the `.dev-bin` path;
    - card 19: a bare-role `efforts` key such as `treadle-judge` with no slash;
    - card 1: the "holds nothing past its call" bound;
    - card 6: the copies clamp line reaching the live report;
    - card 15: `landingDeps` setting `Reap` for a hardener recipe (only the nil rows are planned).

14. **LOW.** Card 10 calls `OnHalt` on every `escalateWith`, including the decision and stalemate causes. D5 names only a campaign halted capped or circling. The wider reach is not stated as a bound in the Intent.

15. **LOW.** Card 20 states "no doc names crucible", but `docs/benchmarks/test-redundancy.md` and several `docs/research/**` files still name crucible, mostly through the `TestMergeCrucible_*` test names. The card neither lists them nor states them as an exempt measurement record.

Tool calls:
- grep `PassOnCap` in `internal/shuttleengine`: my context lacked whether `PassOnCap` passes a failed entry at its cap.
- grep the worktree `_lym/config`, the callers of `loomSegmentBudgets` and `loomrecipe.Routing`, and docs naming crucible or tenter: my context lacked card 21's target path, the budget-projection callers and the docs criterion.
- listed the hub root and grepped `_bolt/_lym/config/gate.yaml`: my context lacked where the hub `gate.yaml` lives.
- read `_bolt/_lym/config/gate.yaml` lines 20-40: my context lacked the tier2 slot setting.
