```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens for round 2: I attacked every bound, guarantee, ordering promise and claim about existing code in `decision-record.md`.

**Claims attacked**
- D8: `Migrate` runs before `pruneSparse` on the hub base file. **Held.** In `configsync.go` `ReconcileHubWideAt`'s default branch, `Migrate` runs before `pruneSparse`. A prime-seeded absent file also migrates through `sparseSeed`.
- D8: the migration runs before the walk's `applyReinterpretations`. **Held.** `walk.go` `reconcileHubWide` calls `ReconcileHubWideAt` first.
- D8: the prime worktree file and the local file are covered. **Held.** `configsync.go` `reconcileSparseFile` runs `Migrate` before `pruneSparse` for both layers, and `ReconcileAll` reconciles both.
- D8: "the migration keeps a hub that set it off, in any layer that set it". **Broken** (F1, F4).
- D8: the dobby starts no pass while one lives. **Held**, but the stated bound for an unregistered pass is too narrow (F3).
- D8: a hand call may pass `--upkeep`. **Held.** `gatecli/testmap.go` `testMapLocation` refuses any worktree but the prime, so a hand call cannot plan a pair's HEAD.
- D8: no pass restarts at the same HEAD without progress (dirty-skip, failed-at-HEAD, spawn-marked, a hub with fewer than two slots). **Held.** A D10 dirty discard can cost at most one extra restart, which the next start's dirty read then skips.
- D8: a pass whose dobby died ends the same way. **Held.**
- D9: a pass takes a slot only when two are free. **Held.** A gate that takes a slot between the count and the take is answered at the next poll.
- D9: "an arriving gate then waits at most one poll behind upkeep". **Broken** (F2).
- D9: "A hand call's wait for a build lock ends within one pass job". **Held.** A yield releases the lock.
- D9: no deadlock between a yielded pass and a hand call. **Held** for the hand call. The pass can still sit idle on a slot (F2).
- D9: "This hub's `gate.yaml` holds `slots: 3`". **Held** (`_bolt/_lym/config/gate.yaml:1`). The template's `slots: 2` claim was not checked.
- D9: the take exists unexported in `pool.go`. **Held** (`tryAcquireAny`, `pool.go:301`).
- D10: the parallel rule selects tests. **Held.** PATTERN-test-speed line 22 has every test open with `tiergate.Require`, and `gateslot/pool_test.go:41-42` follows it with `t.Parallel()`.
- D10: "separate processes are no riskier". **Held.** `lymbin`'s once-per-process build goes into a fresh `lymtemp.MkdirTemp`, so processes do not collide on it.
- D11: `EndDobbies` ends the pass. **Held.** `anchordeps.go:92-96` makes `dobbyOwner(prime)` `runproc.HubOwner`, and `EndSparing` at `anchordeps.go:258` spares only push and sync.
- D11: the pass stops blocking the in-flight check. **Held.** The pass is a direct child of the dobby (`dobbydeps.go:614`, `cmd.Wait`), so exempting its record PID suffices.
- D11: restart after a kill. **Held.** The previous exit record still meets D8's start condition.
- D7: "with no landing, no entry goes newly wrong". **Held.** D6's stamp covers environment change.
- Constraint: a strict load does not refuse a hub that tuned a retired key. **Held.** `configengine/config.go:274` drops retired keys before the fill.
- D6/D1: a rebuild after a stamp miss writes the new stamp. **Unstated** (F5).

**Findings**

**F1 — MEDIUM, design, D8 (off switch).** On a hub that set `test_map_every: 0`, the prime dobby starts passes after a binary upgrade, until a start verb runs the walk.
- Counterexample: a new lym binary is installed. The prime dobby re-execs onto it at its next binary check, and no reconcile runs on that path. Its beam `LoadConfig` goes through `configengine.Load`, whose `DropRetired` silently removes the now-retired `test_map_every`. `test_map_upkeep` is absent, so it defaults to true.
- `hubreconcile.Ensure`, which runs the migration, is called only from `lym beam start`, the batten wire (pair only), the adopt check and clone.
- Evidence: `internal/configengine/config.go:274`; `internal/dobbyengine/doc.go:113`; `internal/beamcli/cli.go:506`; `internal/battencli/wire.go:350`; `internal/adoptcli/check.go:472`; `internal/fabriccli/clone.go:147`.
- Fix: D8 states how the off switch holds before the walk. For example, the beam loader applies the module's `Migrate` in memory, or the dobby's tick runs `Ensure` before it reads the key.

**F2 — MEDIUM, design, D9 (lock retake).** "It takes that lock again with the slot" does not say whether the retake can block, and the yield check does not cover that wait.
- Counterexample on 3 slots: hand call H holds a slot and package P's build lock through its whole rebuild of P. A yielded pass sees two slots free, takes one, then waits on P's lock while holding the slot. A gate G1 takes the last free slot. The pass cannot yield while it waits on the lock, so the next gate G2 waits for H to finish P, not one poll.
- This breaks the D9 rationale "an arriving gate then waits at most one poll behind upkeep".
- Evidence: D9 bullet 2 and its Rationale; `internal/gatecli/testmap.go:466-471`, today's lock take.
- Fix: the retake tries the lock without waiting. When the lock is held, the pass releases the slot and waits under D9's rule.

**F3 — LOW, design, D8 (self-registration).** Moving the `RoleTestMap` registration from the dobby into the verb breaks the dobby's immediate read of a live pass.
- Today `startTestMap` registers the child at spawn (`internal/beamcli/dobbydeps.go:614`), so `testMapRunning` reads it live at once.
- With self-registration, a slow start or a failed registration leaves `TestMapRunning` false. The dobby then starts another pass on each tick. "Refuses to start while another such process lives" cannot see unregistered passes.
- The bound "a pass started beside an unregistered one wastes work" therefore does not cap how many passes start.
- Fix: a pass whose self-registration fails ends before it plans. The dobby does not start a new pass while a pass it started is still its live child.

**F4 — LOW, design, D8 / PATTERN-config-layers.** The migration writes `test_map_upkeep: false` into beam's local override file, but no decision classifies the new key.
- The local layer refuses any key classified other than `CategoryNone` (PATTERN-config-layers background, Layers section). A non-None class would make the migrated local file refused by beam's strict load.
- Evidence: `pattern/PATTERN-config-layers.md:10`; `internal/dobbyengine/config.go:64`, where `test_map_every` is `CategoryNone`.
- Fix: D8 states that `test_map_upkeep` is `CategoryNone`.

**F5 — LOW, consistency, D1/D6.** D1 says a `Commit`-moving write sets the marker's `Commit` and `BuiltAt`. D6 puts the environment stamp in the same marker, but no decision says which write sets it to the current stamp.
- If a whole rebuild after a stamp miss leaves the old stamp, every later plan misses again and rebuilds the package whole at each landing.
- Evidence: D1 paragraph 3 and D6 of the decision record. `internal/testcover` has no marker yet: a grep for "marker" there found nothing.
- Fix: D1 or D6 states that a whole rebuild writes the current stamp.

**Tool calls**
1. `internal/gatecli/testmap.go:200-232` and a grep for `slots:` in the gate config: whether a hand `--upkeep` call can run outside the prime, and the hub's slot count. The manifest held neither.
2. A grep for `sync.Once` in `internal/testkit` and for `tmuxkit.Main`: per-process shared fixtures, to attack D10's "no riskier" claim.
3. A grep for the marker in `internal/testcover`: the format marker's shape, for D1/D6.
4. `internal/testkit/lymbin/lymbin.go` and `boardkit` once-per-process fixtures: whether separate test processes collide on a shared build path.
