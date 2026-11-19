```yaml
lens: guard-reach
outcome: reported
```

Lens: guard-reach. For each guard a card adds, I traced whether a real entry point can reach it and whether the guarded state recurs somewhere unguarded.

**Per guard, reachable or not**
- Card 1, `flagShapeRefusal` (`--copies` &lt; 0, `--copies` without `--gate`, `--copies` with no package): reachable from the CLI.
- Card 1, `gateKeyRefusal` (copies key names several tiers, or tier 3): reachable at call time. Card 5's Preflight does not check that the copies key names a single tier, so it is also reached mid-campaign (finding 2).
- Card 1, inherited-slot refusal: reachable when a call runs under a held `LYM_GATE_SLOT`. Card 14's `hardenLiveEnv` drops that variable for the live gate.
- Card 2, `HubSetsGate` unreadable-file error: reachable.
- Card 3, Guard key in `run_settings` refused at seed, `CheckValues` at seed, `LoadConfig` refusals: reachable.
- Card 4, segment with no Bouncer, build row reached by no edge, segmented order clause: reachable. Parse and Build check no segments (`internal/shedbuild/parse.go:36`, `build.go:29`), so no upstream check pre-empts them.
- Card 5, refusals 1–6: each reachable. `loomCLI.wire` loads strictly before Preflight but checks none of these values.
- Card 6, red mark when 0 binaries started or no started line: reachable.
- Card 6, `Run` error branch for "a gate-config refusal of the call": not reachable as card 14 wires `Run` (finding 1).
- Card 6, tier-3 guard: Preflight only; the live call itself never re-checks (finding 4).
- Card 7, container removed on a failed `NewSandbox`: reachable.
- Card 8, `StopServer` error keeps the entry, `ReapStart` marked interrupted, no `ListProcesses`: reachable. The `EndProcess` step names no caller or ancestor spare (finding 5).
- Card 9, review round fixed as `FixedRound`+1 on resume; error joins with their ways forward: reachable. The "inner error archives nothing" branch fires only on a real error, because `BurlerFix` returns a counted Stuck with a nil error even when a gate fails (`internal/shedadapters/burlerfix.go:326-332`). The card's wording still holds.
- Card 10, `LiveRed` in `settleUndecided` and `softCapRule`: reachable. These are the only two `recordAutoAccept` sites (`internal/shedadapters/bouncer.go:1424`, `:1480`), so no other auto-accept path bypasses the bound. An absent report is read as not red (finding 3).
- Card 10, `OnHalt` on circling, budget or hard-cap escalations: reachable, including the budget escalation the red-live path raises.
- Card 11, `gates`/`gate_key`/non-source fix scope refusals: reachable from a recipe. The nil `Env.Harden` refusal on `HardenFix` is a fail-loud wiring guard (card 14 always fills it). On a review or Bouncer row it is reachable from a recipe without `HardenFix`. The other non-test `shedrecipe.Env{}` builds don't construct Harden rows: batten's is for batten recipes only, and `codeBudgetZero` reads only the Code-Review Bouncer's entry gate (`internal/loomcli/landingdeps.go:650`, `internal/loomrecipe/recipekeys.go:120-131`).
- Card 15, Finalize reap halts Stuck: reachable.
- Card 16, the verbs' recipe refusals and `clear`'s root exception: reachable.
- Card 17, clear-fails-keeps-worktree: reachable.
- Card 19, reinterpretation over the hub base file: reachable. The added way forward covers the `skills` refusal, which the local layer can't reach; the `efforts` refusal, which it can reach, gets none (finding 6).

**Findings**

1. **MEDIUM, design** — card 6 `06-live-gate.md` (`Run` error branch) with card 14 `14-loom-harden-wiring.md` (live `Run` wiring).
   - Card 6 returns "a gate-config refusal of the call" as a wrapped `Run` error and writes no report. Card 14 wires `Run` to spawn `lym gate test` and return only whether it passed and its output, so a refusal exits non-zero and reads as a failed run.
   - On `harden-live`, the refusal output has no started line, so the report goes red and the fixer is re-prompted for a hub config fault it cannot fix.
   - On `harden-live-copies`, the refusal is silently recorded as a copies failure.
   - The card names no way to tell a refusal from a red test run (no envelope or exit-code field on `LiveRun`).
   - Evidence: plan card 6 lines 336–349; card 14 line 811.

2. **MEDIUM, design** — card 5 `05-campaign-preflight.md` (`CheckCampaign` refusals 2–3).
   - Card 1's `gateKeyRefusal` refuses `--copies` over a key naming more than one tier. Preflight checks only that the copies key is set and doesn't name tier 3.
   - So a hub with `harden-live-copies: {tiers: [1, 2]}` passes Preflight and is refused at every round's copies call.
   - This is a guarded state that recurs past the up-front check. Preflight is the place that names the `lym config gate --set` way forward, as it does for tier 3.
   - Evidence: card 1 line 111; card 5 lines 301–303.

3. **MEDIUM, design** — card 11 `11-harden-rows.md` (`Campaign.LiveRed`: "an absent report is not red"), behind card 10's auto-accept bound.
   - The live gate runs after the must-pass `verify` entry, and the gate loop evaluates entries from the first (`internal/shuttleengine/wait.go:537`).
   - When `verify` exhausts, `BurlerFix` returns a counted Stuck with no live run that round (`internal/shedadapters/burlerfix.go:326-332`). No `harden-live-&lt;round&gt;.json` is written.
   - With `auto_accept` set and the budget spent, the Bouncer reads that round as not red and auto-accepts a round Go never verified live, against D6's "no campaign lands past a red live gate without a parent decision" and D4's rule that a run verifying nothing never reads green.
   - The row that fires is the Bouncer's: `settleUndecided` at `internal/shedadapters/bouncer.go:1424`.

4. **LOW, design** — card 6 `06-live-gate.md` (tier-3 bound).
   - "The gate never runs tier 3" rests only on Preflight, which runs once at the run's start.
   - A hub `gate.yaml` changed to name tier 3 for `harden-live` mid-campaign is never re-checked at the live call. Card 1 refuses tier 3 only for `--copies` calls, and the `harden-live` call carries no `--copies`.
   - Evidence: card 6 line 350; card 1 line 111.

5. **LOW, design** — card 8 `08-sandbox-reaper.md` (`Reap` step 4).
   - Step 2 spares the caller for `runproc.End`. Step 4's `EndProcess` kill of unrecorded root-naming processes states no caller or ancestor spare.
   - PATTERN-child-processes has every ending verb spare "the caller and its ancestors". A row's wrapper or a `clear` whose ancestor's cwd or argv names the root would be killed by its own reap.
   - Evidence: card 8 lines 446–448; `pattern/PATTERN-child-processes.md:46-48`.

6. **LOW, consistency** — card 19 `19-treadle-delete.md` (`skillLists` way forward).
   - The added way forward covers only "a `skills` key naming no role". But `skills` is `CategoryGuard`, so a local layer cannot hold it and that state is not reached through `run_settings`.
   - `efforts` is `CategoryNone` (`internal/agentconfig/config.go:122-124`), so a task's `run_settings` can put `efforts: {treadle-judge: …}` into the local layer. The reinterpretation doesn't rewrite that layer, and its load refusal stays without a way forward.
   - The guarded site recurs unguarded on `efforts`.
   - Evidence: card 19 lines 1028–1032.

Tool calls:
1. Grep of `bouncer.go` auto-accept and cap sites, non-test `shedrecipe.Env{` builds, shedbuild segment checks, and `gate.go` error handling — to see whether any other auto-accept path or upstream segment check exists.
2. `internal/loomcli/landingdeps.go:630-675` and `loomrecipe.CodePerchGate` — to see whether that Env build constructs Harden rows with a nil `Env.Harden`.
3. Grep of shuttleengine for gate invocation — to find how a gate error is handled; it returned nothing usable.
4. `internal/shuttleengine/gate.go:30-130` — the `GateEntry`/`PassOnCap` contract for the live entry.
5. The agents module spec, its key categories, and the gate evaluation order in `wait.go` — for card 19's local-layer reach and card 11's absent-report path.
