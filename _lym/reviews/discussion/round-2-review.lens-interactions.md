```yaml
lens: interactions
outcome: reported
```

I reviewed the interactions between the record's decisions. Two pairs are broken: one BLOCKING, one MEDIUM. A constraint also misstates where the scope is computed (MEDIUM).

**Pairs checked:**
- D1 × D3 (coverage removes tests only from importing packages; changed packages run whole): held.
- D1 × D3/D6 (the watch skip acts as under `diff`, but the guard and unrecorded tests always run and the selected invocation carries only `-run`): broken, F2.
- D1 × D6 (tier base from `newestWholeAncestor`; selected entry carries `-run` args): held. `argsMatch` and `newestWholeAncestor` ignore entries with args.
- D1 × D6 (explicit targets are never narrowed; the failed-test rerun from `FailedTestsCall` renders explicit targets): held.
- D2 recorded set × D4/D5 miss domain (both range over *covered* packages only): broken, F1.
- D2 marker × D5 marker miss × Constraint tolerant-records (`WriteMap` keeps an older binary's marker): held.
- D2 failed-build marker × D5 "records a build failure" miss: held.
- D2 dirty-skip and changed-file check × D4 "hashes are the content at `Commit`": held for covered packages.
- D2 staleness on covered drift × D4 compare-from-`Commit`: held. A rebuilt map moves its `Commit`, and D4 compares from it.
- D2 "tests covering only other packages enter the map" × D3 "unrecorded test runs": held.
- D2 redundancy own-package filter × D2 cross-package `Blocks` indexes: held.
- D3 × D4 tier (a test's tier changed in the worktree is an own-file change, a D5 miss): held.
- D3 × D6 (same-name overlap and the over-budget whole run are named in the D3 criterion): held.
- D3 guard tests × the existing guard step (`targetsFor` drops guards of packages in the set; D3 adds them back): held.
- D4 × D5 `init` (same-key functions change together; more `init` functions is a miss): held.
- D4 × D5 new or moved function (moved F: the old file shows F as present only at `Commit`, so it is selected): held.
- D4/D5 × D1 base (map `Commit` before or after the tier base): held. The non-body case is disclosed in Open risks.
- D6 two invocations × `--stop-on-fail` and the failed-test rerun: held.
- D6 selected entry × a later `diff` gate's `covered()` on the same tree: held, no false carry.
- D8 × D2 concurrent testmap rewrite (`WriteMap` is atomic, one read per map): held.
- D8 × Constraint PATTERN-gate-slot ("the scope reads maps inside the gate's own slot"): broken, F3.
- Criteria D1 to D8 against their decisions: held, except the D1 watch-skip criterion against D3/D6 (F2).
- Constraint fabric-boundary × D4: held.

**F1 — BLOCKING, design.** D2 × D3/D5 (`decision-record.md` D2 lines 49–51 and 57–61, D5 lines 118–125). A change to a linked package with no covered block is never seen.
- **Mechanism:**
  - D2 builds with coverpkg naming every linked in-module package. It records files and hashes only per *covered* package.
  - D4 and every D5 miss compare only those covered packages.
  - Go's cover tool instruments function bodies only, so a package reached only through its constants, types or package-level var initializers has no covered block. `fillMapTests` and `PackageBlocks` in `internal/gatecli/testmappackage.go:231-261` key profile blocks, and the profile holds only instrumented function-body blocks.
- **Counterexample:** B's tests read `L.Timeout` (a const) or a struct type of L, and call no function of L. The worktree changes the const.
  - impactset marks L changed and puts B in the `diff` set as an importer.
  - L is not in B's covered set, so D4 finds no changed function and no D5 case fires.
  - B reports `none selected`. The same holds when L is reached through an unchanged function body of a covered package A.
- **Effect:** this narrows silently, against "widen, never narrow", and no Bound names it.
- **Fix:** record and compare every linked in-module package, or every package B imports transitively. A non-body change in one of those is then a D5 miss.

**F2 — MEDIUM, consistency.** D1 × D3/D6 (D1 line 36; D3 lines 75–78; D6 line 136; criterion line 186). The watch skip is not carried into the selected invocation.
- D1 says `//lym:watch` skips act as under `diff`, and the criterion repeats it.
- D3 always adds every guard test and every unrecorded test of an importing package, with no carve-out for the package's unwatched tests (`Targets.Unwatched`, `internal/impactset/targets.go:33-36,154-161`).
- D6 gives the selected invocation only an anchored `-run`. It never says the tier's `-skip` over unwatched tests rides along.
- Today a narrowed share always carries `skip` (`internal/gatecli/scope.go:258`).
- **Fix:** D3's union excludes the package's unwatched tests, or D6's selected invocation keeps the tier's `-skip`. The plan writer should not have to choose.

**F3 — MEDIUM, consistency.** Constraint PATTERN-gate-slot × D8 (`decision-record.md` line 162). The constraint misstates when the scope runs.
- The constraint says "the scope reads maps inside the gate's own slot".
- `computeScope` runs at `internal/gatecli/run.go:82`, before any `AcquireSlots` (`internal/gatecli/run.go:686`, the per-tier acquisition).
- D8's map read happens in scope computation, outside every slot.
- **Fix:** restate the constraint as a read before the tiers' slots, or move map reading into the tier run explicitly.

Tool calls:
1. Grep of `internal/gatecli/run.go` for where `computeScope` runs relative to slot acquisition, and of `//go:embed` lines. My context lacked where the scope runs relative to the slot; the embed grep gave nothing usable.
