```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens. I attacked the record's claims and bounds.

**Claims attacked**
- D4: the live gate holds no slot, so each call acquires its own. **Held.** `LYM_GATE_SLOT` is read only from the environment (`internal/gatecli/run.go:149`).
- D4: an n above the pool is clamped, and a multi-slot acquire never grants part. **Held** (`internal/gateslot/pool.go:280-285`, `pool.go:169-210`).
- D4: `--copies` runs n copies "inside" the call's held slots. **Broken** (finding 5).
- D4: the gate never runs tier 3. **Broken** (finding 4).
- D4: concurrent copies keep tmuxkit's stray check per copy. **Held.** Each binary gets its own lymtemp run directory (`internal/testkit/tmuxkit/tmuxkit.go:27-44`).
- D4: the `--copies` bound ("needs `--gate`, holds no slot past its call"). **Broken** as an attack surface (finding 6).
- D2: Preflight detects a hub that "sets no `harden-live` key of its own". **Broken** (finding 7).
- D3/D2: the adjacency refusal makes CONVERGED reachable. **Held** for plain specs (including first equals last). Weak for effort-qualified specs (finding 10).
- D5: the reaps touch only servers whose key derives from the hub list. **Broken** under concurrent `new` calls (finding 8).
- D5: fabric's leftover cleaner refuses a path holding `.git`. **Held** (`internal/fabricengine/destroy.go:1426,1451`).
- D5: the teardown bound (Finalize, plus the capped or circling halt). **Broken** (finding 3).
- D5: the hub list outlives a hub because `reedengine.ServerName` hashes the path. **Held** (`internal/reedengine/server.go:47-52`).
- D5: recorded processes end through `runproc.End`; `EndProcess` ends unregistered ones. **Held** (`internal/runproc/end.go:60,135`).
- D6: a red live gate ends capped and the parent decides. **Broken** (finding 2).
- D7: the Bouncer's seam runs only on an approved settle. **Held.** Auto-accept also goes through `approveAndCommit` (`internal/shedadapters/bouncer.go:550-557,1371`).
- D8: `agentconfig.Load` refuses an unknown role. **Held** (`internal/agentconfig/config.go:161-165,217-219`). The reinterpretation's reach is broken (finding 9).
- D9: "a task's `run_settings` may override any of them". **Broken** (finding 1).
- D10: today's build rows carry no segment, so they are checked as before. **Held.** No Implementer, DarnWrite, BeamAwait or SamplerSession row in `contracts/recipes/*.yaml` sets a segment.
- D3: pickRound clamps; D1: batten's child-recipe list is closed. **Held** (`internal/burlerengine/models.go:26-31`, `internal/battencli/wire.go:606`).

**Findings**

1. **BLOCKING, design.** `_lym/discussion/decision-record.md:216` (D9), with D2 at `:49`.
   - Summary: `run_settings` reach a run only through the pair's local layer, and PATTERN-config-layers lets that layer hold only CategoryNone keys.
   - Counterexample: `auto_accept` is Guard (`internal/reviewbudget/reviewbudget.go:42`). The fans are Guard (`internal/loomengine/config.go:649-656`). darn's `tree_scope` is Guard and "no task run_settings sets it" (`internal/darnengine/config.go:59`). A task's override of any of these is refused.
   - D2 needs `packages` and `llm_driving` open to the local layer. Yet `packages` decides what the live gate checks, so if it is classified None, a local file can narrow the gate to a trivially green package.
   - D9 classifies no key and makes a promise the PATTERN forbids.

2. **MEDIUM, design.** `decision-record.md:170-171` (D6) against D9's `auto_accept` at `:210`.
   - Summary: "A live gate left red … blocks CONVERGED, so such a campaign ends capped and its parent decides" does not hold with `auto_accept` on.
   - Counterexample: the Bouncer turns a spent budget into an automatic accept (`internal/shedadapters/bouncer.go:260,1371`). The run then goes on to Describe, PR-Review and Finalize with the live gate red and no parent decision.
   - The record states no bound on `auto_accept` for this segment.

3. **MEDIUM, design.** `decision-record.md:142-145` (D5 teardown bound).
   - Summary: the reap runs only at Finalize and at a capped or circling halt, but `lym fabric remove` and batten's Worktree-Teardown also end a pair through `internal/pairteardown` (`internal/pairteardown/doc.go:1-5`). They run no reap.
   - Counterexample: an operator removes an abandoned or halted hardener pair.
   - The sandbox hubs' dobbies and reed servers are recorded in each sandbox hub's own runproc directory, not under the pair's slug, so the pair's process end misses them and they survive.
   - The root's nested `.git` then keeps `ignoredTaskPathDeletable` refusing the leftover path (`internal/fabricengine/destroy.go:1451`).

4. **MEDIUM, design.** `decision-record.md:96-97` (D4 step 3).
   - Summary: "It never runs tier 3" rests on Preflight's one check at run start.
   - Gate keys are hub-file config that can change mid-campaign. A plain `harden-live` call has no tier-3 refusal; only `--copies` has one.
   - `Config.Gate` accepts any declared tier (`internal/gateslot/doc.go:199-201`), and `manual: true` keeps a tier out of `--upto` only (`doc.go:148`).
   - Counterexample: `lym config gate --set` gives `harden-live` the tiers `[2, 3]` after Preflight. The next round's live gate runs billed tier 3.

5. **MEDIUM, design.** `decision-record.md:107-110` (D4 `--copies`).
   - Summary: "acquires n slots for the call and runs n concurrent copies inside them" is not a mechanism the gate has.
   - A tier run takes its own slots through `acquireSlot` → `AcquireSlots` (`internal/gatecli/run.go:737`). It skips that acquire only when `LYM_GATE_SLOT` in its environment names a held slot (`run.go:149`). So an in-process n-slot hold followed by the tier's own acquire contends with itself.
   - "Each named package's compiled test binary" also departs from the tier's shell command (`go test -json {test_targets} {args}`, hub `gate.yaml` tier2) and from its go-json counter.
   - The record states neither how the tier's acquire is replaced nor how copies are started from the tier command.

6. **LOW, design.** `decision-record.md:108` (D4 `--copies` bound).
   - Summary: the flag goes with any key and any caller, including the tool-use reviewer.
   - Counterexample: `--copies 50` on `--gate harden-live-copies` takes the drain lock and every hub slot (`internal/gateslot/pool.go:179-210`) for up to tier 2's `timeout_min: 30`. Every other worktree's gates queue behind it for that time.
   - The bound limits neither the keys nor the callers.

7. **LOW, design.** `decision-record.md:54-55` (D2 Preflight).
   - Summary: "the hub's own `gate.yaml` sets no `harden-live` key" cannot be read through the loaded Config, which fills the template's `gates` entries (`internal/gateslot/doc.go:103-104,113-114`). The check needs a base-layer read, which PATTERN-config-layers reserves to configengine.
   - A hub whose live tier equals the template's `[0]` holds a template-equal unit that hubreconcile's one-time strip may drop. Preflight then refuses with a way forward that writes nothing that lasts.

8. **LOW, design.** `decision-record.md:128,139` (D5 hub list).
   - Summary: a round agent runs several `sandbox new` calls at once, since a concurrency claim needs a second hub. The record names no lock on the hub list's append.
   - A lost update leaves a hub unrecorded, and its reed server falls outside "touch only reed servers whose socket key derives from a hub path the root's hub list records".
   - PATTERN-lym-files gives lock paths through `lock`.

9. **LOW, design.** `decision-record.md:195-196` (D8).
   - Summary: the reinterpretation rewrites only the hub base file (`internal/hubreconcile/reinterpret.go:50-58`).
   - `efforts` is CategoryNone (`internal/agentconfig/config.go:124`), so a `treadle-judge/&lt;model&gt;` key can sit in a task's `run_settings`, in `profiles.yaml` or in a pair's local agents.yaml.
   - None of these is rewritten. `checkEfforts` refuses such a key once the role is deleted (`config.go:217-219`), so `agentconfig.Load` fails for that pair.

10. **LOW, consistency.** `decision-record.md:51` (D2), with D6 at `:170`.
    - Summary: Preflight's "one model in two adjacent rounds" and the judge's "a reviewer model different from the previous round's" are not tied to one comparison.
    - Counterexample: the list `[opus[high], opus[low], sonnet]` passes a spec-string adjacency check, yet one model reviews rounds 1 and 2.

Tool calls (4):
- Grepped `contracts/recipes/*.yaml` for segments on build rows, to test D10's "checked as today" bound; the manifest had no recipe YAML.
- Read `internal/loomrecipe/darnrecipe.go:20-60` and grepped AutoAccept and Commit in `internal/shedadapters/bouncer*.go`: one hop for the other reader of `LoomBuildEngines`, and for whether auto-accept settles through the commit seam.
- Grepped teardown routes and read `internal/pairteardown/doc.go`, to find pair-ending paths outside Finalize for D5's bound.
- Grepped `internal/testkit/tmuxkit/tmuxkit.go`'s doc for its private directory scope, to test whether concurrent copies collide.
