```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced every existing code path that reads or writes the state D1 to D8 introduce or change.

**Findings**

1. **BLOCKING, D2 and D5: if the cross-package hashes go into the map's `Files`, `lym gate testmap` rebuilds every map on every run.**
   - Evidence: `testMapRun.stale` in `internal/gatecli/testmap.go` returns stale when `!maps.Equal(m.Files, files)`, where `files` holds only the package's own Go files at HEAD.
   - D2 says the dependency-file hashes are recorded "beside its own package's files", but never says where they live or how `stale` treats them.
   - If they share `Files`, every rebuilt map compares unequal, so the backlog never drains.
   - `mapDrift` in `internal/gatecli/redundancy.go` reads `Files` too, and would report foreign files as drift.
   - Fix: D2 states that the dependency hashes go in a separate field, and that `stale` and `mapDrift` keep comparing own-package `Files` only.

2. **MEDIUM, D5 against testmap's own staleness: a map that misses because a dependency drifted stays a miss until the age limit.**
   - Evidence: `stale` checks only own-package `Files`, the age and a build failure.
   - D5 makes a map a miss once any recorded dependency-file hash differs from the base, but the existing refresh path never rebuilds a map for that reason.
   - So a map can stay a scope miss for up to `test_map_max_age_days` (7) after one landing in a widely imported package, which is exactly the package this task targets.
   - The Open risks line blames freshness on `testmap-upkeep` but never says the existing staleness rule ignores the new field.
   - Fix: decide whether `stale` also compares the dependency hashes against HEAD, or state the gap as a bound under D5 and Open risks.

3. **MEDIUM, D1: `coverage` takes effect only through the `--gate` path.**
   - Evidence: in `internal/gatecli/scope.go`, `withGateScope` switches on `cfg.Gates[request.gate].Scope` with `case "all"` and `case "diff"`. Any other value falls through and keeps the computed form.
   - `internal/gateslot/tiers.go` prints `key.Scope` in the gate listing.
   - The record gives no way for the computed form to learn it is coverage-narrowed.
   - Explicit targets, including the failed-test rerun from `FailedTestsCall` and Darn's `tree_scope` command, bypass the key's scope entirely.
   - D1's claim that "everything `diff` does … `tree_scope`" runs first holds only because `tree_scope` replaces the scope with explicit targets. It is then never narrowed, which D1 should state.
   - Fix: D1 names that `withGateScope` must carry `coverage` as a computed form with a narrowing flag, and that `tree_scope` and explicit-target calls stay un-narrowed.

4. **MEDIUM, D6: the green record's carry logic already matches some `-run` args.**
   - Evidence: `greenRecord.covered` in `internal/gatecli/green.go` matches an entry with empty args to a request with empty args, to the guard step's own `-run` filter and to the watch skip. Any other args must match exactly.
   - `greenRun.subtract` drops packages already green on the same tree.
   - Read path: a selected-test invocation whose packages already passed whole on the same tree gets subtracted, which is correct.
   - Write path: D6 says a selected invocation records no entry that marks a package passed. The record should say whether it records an args-carrying entry or none at all. An args-carrying entry is already excluded from `newestWholeAncestor` because its args are non-empty.
   - The record is silent on whether the new invocation's anchored `-run` gets the guard filter's empty-args matching. If it did, a whole pass would carry it, which is fine. If it were treated like an empty-args request, a selected pass could carry a later whole run, which is wrong.
   - Fix: D6 states that the selected invocation's entry carries its `-run` args, so it never matches an empty-args request.

5. **LOW, D2 and the redundancy report: the "own-package only" filter is needed only for kept tests.**
   - Evidence: in `internal/gatecli/redundancy.go`, the kept set comes from `p.m.Tests` mapped through `p.m.Blocks[index].String()`. Fresh runs are keyed from a binary built with coverpkg set to the package alone.
   - Extra foreign keys among kept tests cannot change a verdict on own-only fresh runs, since kept tests are never judged.
   - So the D2 rule holds, but the filter belongs on the kept side, or the fresh build's coverpkg must stay own-package.
   - Fix: D2 says the redundancy step's fresh build keeps an own-package coverpkg, or filters both sides.

6. **LOW, D2: maps that `writeFailed` writes are not covered by the marker rule.**
   - Evidence: `internal/gatecli/testmappackage.go` sets `m.Blocks, m.Tests = nil, nil` for a build failure.
   - D5 treats a build failure as a miss whether or not the marker is set.
   - D2's "a map without the marker is stale to testmap" would make a failed map at HEAD rebuild on every refresh, whereas today it waits for a new commit.
   - Fix: D2 states that `writeFailed` also writes the marker.

7. **LOW, D1: template prose and the config error string.**
   - Evidence: `internal/gateslot/template.yaml`'s comment lists "webster-fix, darn-write, code-burler-fix and pr-review (scope diff …)".
   - The validation message in `internal/gateslot/config.go` reads "want card, diff or all".
   - The acceptance criteria cover the error. The template comment needs a `Review:` criterion.

**Paths traced, per decision**

- D1:
  - `withGateScope`: breaks unless extended (finding 3).
  - `decodeGates` validation: covered by an acceptance criterion.
  - Retired-list migration `gateTarget` in `config.go`: survives, since it writes card and diff only.
  - `run.go`'s card check: survives.
  - `tiers.go` gate listing: survives.
  - Darn's `darnTreeScope`: survives, un-narrowed.
  - `FailedTestsCall` rerun: survives, as explicit targets.
- D2:
  - `testMapRun.stale`: breaks (finding 1).
  - Redundancy `mappedPackages`, `checkMapBlocks` and kept-set build: survive (finding 5).
  - Redundancy `mapDrift`: breaks if hashes share `Files` (finding 1).
  - `writeFailed`: unclear (finding 6).
- D3 and D4: `computeScope` with `impactset.DeriveTargets` survives. Coverage narrows after derivation, and the guard and watch steps are separate.
- D5: testmap refresh drives freshness and never clears the dependency-hash miss (finding 2).
- D6:
  - `greenRun.subtract` and `covered`: survive if the args are recorded exactly (finding 4).
  - `newestWholeAncestor`: survives, since the entries carry args.
  - `--stop-on-fail` and the failed rerun: survive.
- D7: no existing reader.
- D8: `ReadMap` is the only loader, and testmap writes under its build lock. A torn read reads as unreadable, so it survives.

**Tool calls**

1. Grepped for `"diff"` and scope constants: to find existing switches on the scope value, which my context lacked.
2. Grepped `ReadMap`, `.Tests`, `JudgeAgainst` and testmap staleness: to find every map reader and the `stale` rule's body.
3. Read the `greenEntry` fields and `covered`, and grepped the green functions: to learn how args and packages match in the carry logic.
4. Grepped scope-value uses in `gatecli`, `gateslot` and `loomcli` and `darnwiring` scope handling: to trace who reads a key's scope.
5. Grepped the `withGateScope` switch and the `key.Scope` readers: to confirm how an unknown scope value falls through.
