```yaml
lens: repo-invariants
outcome: reported
```

Task: run the repo-invariants lens over cards 1–23, covering test seams and tier tags, package docs and PATTERN text the cards make false, and card order.

**Per card, each invariant checked**

- **Card 1:** no shared seam struct; `TestConfig_PushTiming` is tier0; `fabricengine/doc.go` not affected; order held.
- **Card 2:** no reflective test covers the `sheddriver.Deps` fields it adds; the card edits `sheddriver/doc.go`, `loomcli/doc.go` and the reflection template. `pattern/PATTERN-shed-driver.md` is no longer accurate, and no card lists it (LOW, below). Order held: it uses card 1.
- **Card 3:** tier0 tests. `dobbyengine/doc.go`'s run-watch section no longer matches, and the card lists that doc only under Uses: broken (finding 2).
- **Card 4:** the new test package `internal/pushledger` gets no `TestMain`: broken (finding 1). The tier is stated, and the `lymdirs` doc and `docs/overview.md` are extended: held.
- **Card 5:** the `NewDeps` re-sign, the `pusherDeps` and `runpusher/doc.go` edits, `unboundedWaitFiles` and the internal `TestNewDeps_*` test are all listed: held.
- **Card 6:** order held after card 5; `beamcli/doc.go` is listed under Uses only, but the restarts are not described there in a way that becomes wrong: held.
- **Card 7:** the docs it makes false are all listed. `PATTERN-fabric-boundary.md:90` ("detached `SpawnPairPush`, never a synchronous push") still reads true: held.
- **Card 8:** held.
- **Card 9:** `pairteardown/doc.go:168-174` says `PushRun` pushes the run's records, with no hold. That doc is listed only under Uses: broken (finding 3). `beamcli/doc.go:103,108` say `lym beam pr` pushes the beam branch, which `CodeCeiling` now cuts short: broken (LOW, finding 5).
- **Card 10:** held.
- **Card 11:** `pairteardown/doc.go`'s hand-over paragraph (via card 19): see finding 3. Order held: it edits `TargetRefused` and `restartHubPushers`, which card 6 creates.
- **Card 12:** the `landingshed.Deps` field removals are reflected in `envkit#nilLegal`. `envkit.LandingDeps` (`envkit.go:194-262`) sets none of the removed fields, and `TestLandingDeps_EveryFieldPopulated` is under Uses: held. Deleting `parentMergeArchiver` may leave `fabricengine.ArchiveParentMerge` with no caller: unverified (LOW, finding 6).
- **Card 13:** held.
- **Card 14:** `MergeInBasePath` is a string field, and the `IsZero` check at `landingdeps_test.go:146` needs `landingDeps` to fill it. The card wires it there: held.
- **Card 15:** held.
- **Card 16:** held.
- **Card 17:** `dobbyengine/doc.go`'s catch-up section (lines 240-252) still reads true at engine level: held.
- **Card 19:** `pairteardown/doc.go:174` ("Unless the caller runs inside the pair, it removes the local pair") becomes false once `HandOver` skips the removal while a landing stands. The `ErrLandingStands` refusal in the teardown composite is undocumented, and the doc is listed only under Uses: broken (finding 3). `battenshed/doc.go` is edited: held.
- **Card 20:** the new per-tick summary step gets no description in `dobbyengine/doc.go`, which is listed only under Uses: broken (finding 2).
- **Card 21:** tier0 tests; the reed doc and the `bar.go` file comment are named: held.
- **Card 22:** held.
- **Card 23:** the backgrounds the decision record names are covered. `PATTERN-shed-driver.md` is not among them (finding 4).

**Findings**

1. **BLOCKING · consistency · card 4 (`04-push-ledger.md`).**
   - The new package `internal/pushledger` gets `ledger_test.go` (and card 21's `TestSyncStatus`) but no `testmain_test.go` calling `tmuxkit.Main`.
   - The tier0 guard `TestTmuxIsolation_EveryTestPackageRunsThroughTmuxkitMain` fails every test package without one (`cmd/lym/tmuxisolation_test.go:261-276`), so `PATTERN-test-isolation` is broken.
   - Fix: add `internal/pushledger/testmain_test.go` to card 4's Create.
2. **MEDIUM · consistency · cards 3 and 20 (`internal/dobbyengine/doc.go`).**
   - The run-watch section (`doc.go:153-172`) defines a stop as any batten or child state other than running. Card 3 makes a `Waiting` run owe no line and a `WaitDied` run one stop.
   - Card 20 adds a per-tick push-summary step, which the doc never describes.
   - Both cards list `dobbyengine/doc.go` only under Uses, so neither edits it.
   - Fix: move it to Edit on cards 3 and 20.
3. **MEDIUM · consistency · cards 9 and 19 (`internal/pairteardown/doc.go`).**
   - `doc.go:168-174` says `PushRun` pushes the run's records whole, and that `HandOver` removes the local pair unless the caller runs inside it.
   - Card 9 adds the records hold to `PushRun`. Card 19 has `HandOver` skip the removal while a landing stands, adds the `ErrLandingStands` refusal to `Run`, `RemovePair` and `EndForCancel`, and has `RemovePair` drop the ledger entry.
   - Both cards list the doc only under Uses.
   - Fix: move it to Edit on cards 9 and 19.
4. **LOW · consistency · card 2 / card 23 (`pattern/PATTERN-shed-driver.md:37`).**
   - "It ends in the tackler under its spawn budget, a re-step under the repair cap, or a hand-back" no longer holds for network-class stops, which card 2 re-steps with no cap.
   - Card 23 does not list this background.
   - Fix: add it to card 23.
5. **LOW · consistency · card 9 (`internal/beamcli/doc.go:103,108`).**
   - The doc says `lym beam pr` pushes the beam branch. Under card 9, `PushBeam` goes through `PushRun` with `CodeCeiling` and stops short of standing landings.
   - No card edits the doc; card 9 does not name it.
6. **LOW · consistency · card 12.**
   - Card 12 deletes `loomcli#parentMergeArchiver`, the caller of `fabricengine.ArchiveParentMerge` I found (`internal/loomcli/landingdeps.go` ~918), and no card deletes or names `ArchiveParentMerge`.
   - If it has no other production caller, `PATTERN-test-only-code`'s reachability rule breaks. I did not check for other callers.

**Tool calls**

1. `grep` for reflective tests, envkit's seam walk, and the `TestMain` scans in `cmd/lym`: the manifest had no seam-test inventory.
2. Read `envkit.LandingDeps`, the `IsZero` and `Kind` checks, and `cmd/lym/testmain_test.go`: needed to judge the cards 12 and 14 seam impact and the `TestMain` rule.
3. Read the hermetic-env and tier-tag test headers, and grepped `doc.go`, `pattern/`, `docs` and `PATTERN.md` for wording about the old push, teardown and re-step behaviour: doc text was not in the manifest.
4. `grep` of `pattern/` outside card 23's list, and of the batten, dobby, pairteardown, beam, fabriccli and landing `doc.go` files, for push and teardown wording: same reason.
5. Read the `dobbyengine/doc.go` run-watch and catch-up sections and `pairteardown/doc.go` lines 160-185: needed to confirm the text those cards make false.
6. Read `pattern/PATTERN-shed-driver.md` lines 28-40 and the tmuxkit-isolation scan: needed to confirm finding 4 and the scope of finding 1.
