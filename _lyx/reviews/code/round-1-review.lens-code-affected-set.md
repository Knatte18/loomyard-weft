```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: I found no finding. Everything the diff changes is covered, and nothing outside the touched lines is left wrong.

**Per change, the set found:**
- **`drivePasses` filter on `d.runs`:** The readers are `recordOwed`, the pass-planning loop, the `listed` map for `liftPending`, `triggerFor` and `runsInFlight` (`starts.go:103-105`, `startReady:66`).
  - All of them are fed from the filtered slice, so they are covered.
  - `originOwed` (`passes.go:313`) calls `deps.Runs()` unfiltered, but it reads only `OriginOwed` of the run whose pass just exited. A done run's pass in flight still gets that read, so this is correct.
  - `collectExits` runs before the filter. The `passExited` entry it writes for a done run whose pass just exited is dropped by `recordOwed` in the same tick, so it is covered.
- **`FullPassOwed` and `OwnedElsewhere` writes for a done run:** `owned.go:21` writes them only for listed runs, and a done run is no longer listed after its pass exits. The `FullPassOwed` reader in `battencli/arm.go:650` reads batten's own state, not dobby's `state.json`. Both are covered.
- **`Runs` seam doc (`loop.go:63-65`):** It says the seam lists the parent's runs. The seam still does, and the engine filters afterwards, so the doc is not stale.
- **Board tidy and `Landings`:** `countLandings` and the `Landings` seam (`loop.go:88`, `tidy.go:98-105`) don't touch `d.runs`. The landing count is unchanged, as the board entry asks. The `beamcli` listing is also untouched.
- **Docs:** `doc.go` (passes and starts bullets) and the `startReady` comment in `starts.go` are updated. `template.yaml:1` ("the most runs … dobby drives at once") and `config.go` still read correctly. `PATTERN.md:35` and `docs/overview.md` carry no `max_runs` or in-flight wording, and the invariant is unchanged, so no doc is owed. No stencil mentions this behavior.
- **Tests:**
  - `passes_test.go` changed only to add the `stateRecord` helper.
  - `stateOf` still returns the same value, and I found no other caller of the changed helper.
  - Whether the new subtests pin the behavior is another lens's call.

**Tool calls (2):**
1. `grep` for `max_runs`, `d.runs` and `Landings` across the repo: my context lacked the non-engine readers.
2. `grep` for `Runs func`, `full_pass_owed` and dobby mentions in `PATTERN.md` and `docs/overview.md`: my context lacked the seam doc and the doc set.
