```yaml
lens: way-forward
outcome: reported
```

Way-forward lens report.

**Findings**

1. **MEDIUM**, card 1 (D1 package set) with card 5 (D9): `lym gate redundant`'s "no map" refusal can loop.
   Card 5 sends a package with no map to `lym gate test --map <package>`. Card 1 admits a package into the map set only when it "has a map … or holds a tier0 or tier1 test", and leaves any other package out silently with exit 0.
   For a real package that holds Go files but no tier0 or tier1 test, the map run writes nothing and exits 0, and `redundant` then refuses again with the same way forward.
   Evidence: `_lym/plan/01-gate-test-map-mode.md` Intent ("every other package … is left out silently") against `_lym/plan/05-redundant-map-way-forward.md` Intent (`rebuild` for a missing map).
   Fix: list each explicit target the set leaves out, with its reason (holds no tier0 or tier1 test), in the envelope or as a refusal, or give `redundant`'s missing-map refusal a branch for a package with no mapped test.

2. **MEDIUM**, card 1 (D1 counter refusal): the way forward can only clear the refusal by breaking the hub.
   The text tells the user to "set tier<n>'s counter to go-json with lym config gate". On a hub whose tier0 or tier1 counts with `vstest`, that is a .NET tier, and retyping its counter breaks the tier's real runs. Nothing on such a hub can build a map.
   Evidence: `_lym/plan/01-gate-test-map-mode.md`, the `testMapRun.prepare` paragraph. `internal/gateslot/tiers.go:247-248` admits `go-json` and `vstest` only, and `internal/gatecli/counters.go:23` holds `counterGoJSON`.
   Fix: name the actual end, that a coverage map exists only for a go-json tier so this hub cannot build one, rather than a config change that misconfigures the tier.

3. **LOW**, card 4 (D6 unmeasured list): the unmeasured reasons name no next step.
   - "HEAD moved from <start> to <now>" does not say to re-run once HEAD settles.
   - "modified or untracked file <path>" does not say to commit or revert the file and re-run.

   The call exits 1 with only these entries as guidance. Card 5's stale-map refusal does tell the user to commit or revert first, so the two cards are inconsistent.
   Evidence: `_lym/plan/04-map-run-clean-checks.md`, the `checkClean` bullet.
   Fix: append the way forward to each reason, or add it once beside the `unmeasured` key.

4. **LOW**, card 1 (`flagShapeRefusal` ordering): `--map --tags x` takes a detour through two refusals.
   The map refusals are checked after the `--tags` refusal. That refusal's way forward is "name the tier … with --tier, for example --tier 1", and the new `--map` refusal then refuses `--tier`. The user ends up at plain `--map` only after both.
   Evidence: `internal/gatecli/run.go:341-343` against `_lym/plan/01-gate-test-map-mode.md`, the `flagShapeRefusal` bullet.
   Fix: refuse `--map` with `--tags` in the map block, telling the user to drop `--tags`, or check the map partners before `--tags`.

5. **LOW**, card 1 (D9, the retired verb): `lym gate testmap` now answers only cobra's `unknown subcommand "testmap" for "gate"`.
   That message does not point a habitual caller to `lym gate test --map`. The decision record asks only for an unknown command, so this is a usability gap, not a breach.
   Evidence: `_lym/plan/01-gate-test-map-mode.md`, the `TheRetiredVerb` row.

Cleared: map mode cannot trip the detached runner's cold card fixer, even though card 1's "never reaches the card fixer" is claimed for `runTest` alone. `handsOff` turns the fixer on only for a request that carries `--gate` (`internal/gatecli/cardfixer.go:83-86`), and card 1's `flagShapeRefusal` refuses `--map` beside `--gate`.

**Per refusal or stop: next step, and whether it progresses**
- `--map` with `--tier`, `--upto`, `--gate`, `--guards`, `--stop-on-fail`, `-C` or words after `--`: drop the flag and use targets or a scope flag. Progresses.
- `--map` with `--tags`: add `--tier`, which is then refused, then drop it. Progresses after a detour (finding 4).
- `--map` with `--main-packages` and no `--card`: the existing refusal says to pass `--card` or drop the flag. Progresses.
- Several scope forms: keep one. Progresses.
- Outside a hub: run from a worktree of a hub. Progresses.
- Target naming no package directory: name a module-relative package. Progresses.
- Tier counter not go-json: set it to go-json. Does not progress safely (finding 2).
- Tier `memory_mb` below the test cap: raise `memory_mb` or lower `redundancy_test_memory_mb` with `lym config gate`. Progresses.
- Cancelled call or signal: re-run the same command. Progresses.
- Slot wait under `slots: all`: waits for the other holders to release. Progresses.
- HEAD-moved end (card 4): no step named (finding 3).
- Dirty package unmeasured (card 4): no step named (finding 3).
- Failed listing with a dirty module file (card 4): no step named (finding 3).
- Exit 1 with `failed_tests`: fix the tests, which the list names. Progresses.
- `redundant` with no map: rebuild with `--map`. Loops for a package with no tier0 or tier1 test (finding 1).
- `redundant` with an unreadable map: `--map` replaces the map whole. Progresses.
- `redundant` with a map holding only a failed build: `--map` rebuilds. Progresses once the build is fixed, and the refusal shows the build error.
- `redundant` with a map built from other files: commit or revert, then `--map`. Progresses.

**Tool calls**
- Grepped `detachrun.go` for `runWithFixer`: my context did not have the detached runner's fixer wrapping.
- Grepped `detachfixer.go` for `runWithFixer`'s conditions: I needed to know when a red map run would get a fixer.
- Grepped `cardfixer.go` for the fixer seam: I needed to find `newCardFixer`.
- Read `cardfixer.go:130-179`: to read `newCardFixer`'s on-conditions.
- Grepped for `handsOff`: to confirm that the fixer requires `--gate`.
