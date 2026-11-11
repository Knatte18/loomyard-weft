```yaml
lens: walkthrough
outcome: reported
```

Task: walkthrough lens. For each decision, run one concrete case through its mechanism and check whether the end state meets the decision's purpose.

**Walkthrough list**
- **D1.** Case: a landing changes the body of F, which test T covered; on a later pass the verdict is Current and an own Go file differs. Purpose met for Affected and Miss. The Current text contradicts D4's own-file drop (finding 3).
- **D2.** Case: a helper edit in a test file, then an edit to a `Test` function's text. Met: the first is a miss, the second selects that test.
- **D3.** Case: test T of package B calls F of package A. Met: the map records F under A's import path and no block of A.
- **D4.** Case: T fails alone; then a landing changes B's own non-test file while the verdict stays Current. The outcome entry is dropped by D4 but kept by D1's text (finding 3). Spawn-marked tests: met.
- **D5.** Case: an Affected plan moves the map to H, then the build at H fails, then the next pass runs at the same H. Met: D8's exclusion keeps the package off the queue, so D1's gap queue does not restart it.
- **D6.** Case: a toolchain bump. Met: the stamp differs, so the gate misses and upkeep rebuilds.
- **D7.** Case: a pass whose plan finds every map current. Met on a hub with two or more slots; on a hub with fewer the audit never runs (stated through D9's bound).
- **D8 trigger.** Case: a landing moves HEAD from H0 to H1, a pass runs, and later a pass is killed. Met: the restart condition holds whether or not a killed pass writes the exit record.
- **D8 off switch.** Case: a hub base file holds `test_map_every: 0` and a new binary is installed. Broken in the window before the walk runs (finding 1).
- **D8 local layer.** Case: a local `beam.yaml` holds `test_map_every: 0`. The migration runs before the prune. Whether the migrated file then loads depends on the new key's category, which the record leaves open (finding 4).
- **D9.** Case: 3 slots, one gate holding a slot, the pass takes one, a second gate takes the last free slot. Met: the pass yields within one poll. Case: 2 slots, a hand call waits on the lock the pass holds. Met: the pass sees every other slot held, yields and releases the lock.
- **D10.** Case: a Miss's whole rebuild that writes round 1 and then yields. The record does not settle whether the first round's write drops the old entries (finding 2).
- **D11.** Case: an anchor move of the prime while a pass runs, including with the dobby dead. Met: `dobbyWorktrees` always names the prime, and `EndSparing` over `HubOwner` with only push and sync spared ends the pass's record.
- **D12.** Case: a pass ends with queued work. Met: the dobby reads the planned HEAD and the queued count.
- **D13.** Case: carrying D2 onto `coverage-test-select`. Met.

**Findings**

1. **MEDIUM, design. D8 (off-switch migration).**
   A hub that set upkeep off gets passes after a binary upgrade, until a start verb runs the walk.
   - The running prime dobby re-execs onto the new binary between ticks and reconciles no config (`internal/dobbyengine/doc.go:113`).
   - The only `hubreconcile.Ensure` callers are `lym beam start`, adopt check, clone and the batten pair wire (`internal/beamcli/cli.go:506`, `internal/adoptcli/check.go:472`, `internal/fabriccli/clone.go:147`, `internal/battencli/wire.go:350`).
   - The dobby's strict `LoadConfig` goes through `configengine.Load`, which silently drops retired keys (`internal/configengine/config.go:274` `DropRetired`). With `test_map_every` retired, `test_map_upkeep` falls back to its default, true.
   - So on its first tick on the new binary, the dobby sees no exit record and starts a pass on a hub that set `test_map_every: 0`. D8's claim that "the migration keeps a hub that set it off, in any layer that set it" fails for that window. Neither D8's Bound nor its criterion covers it.
   - Fix: D8 states how the dobby (or `--upkeep`) reads the off switch before the walk has run, for example by applying beam's `Migrate` in memory at load. Otherwise D8 states the window as a bound.

2. **MEDIUM, design. D1 Miss with D10's per-round writes and D9's re-plan.**
   - D1's Miss "leaves the map as it is and queues a whole rebuild". D10 then has the job write the map after each round, at H.
   - The record never says that a whole rebuild's first write starts from an empty map. If the write merges finished runs into the existing map and moves `Commit` to H, the stale entries are carried to H.
   - D9's re-plan after a yield, or the next pass, then reads that map as Current at H and keeps those entries. The gate stops running tests whose coverage the Miss had just declared untrusted.
   - Fix: D1 or D10 states that a whole rebuild's first written round replaces every entry, so the map at H holds only entries measured at H.

3. **LOW, consistency. D1 Current against D4.**
   - D1's Current verdict says "every entry is kept" (`decision-record.md:35`).
   - D4 drops an outcome entry "when any of the package's own files differs between the map's `Commit` and `H`" (`decision-record.md:102-104`). That can happen under Current: D2 compares own Go files by function, so an own-file change can leave no test affected.
   - Fix: D1's Current bullet excepts D4's outcome-entry drop.

4. **LOW, design. D8's new key `test_map_upkeep`.**
   - D8 has `Migrate` write `test_map_upkeep: false` into beam's local override file (`decision-record.md:180-181`).
   - The local layer refuses every key whose category is not None (`pattern/PATTERN-config-layers.md:10`), and beam's loader is strict.
   - The record gives the new key no category. If the plan writer gives it anything but None, the migrated local file fails every beam load.
   - Fix: D8 states that `test_map_upkeep` is `CategoryNone`, as `test_map_every` is (`internal/dobbyengine/config.go:64`).

Tool call:
- `internal/fabriccli/anchor.go:360-385` and `internal/fabriccli/anchordeps.go:175-200`, for which dobbies an anchor move ends (`dobbyWorktrees`). My context had the call site but not the definition, and D11's walk with a dead dobby depended on it.
