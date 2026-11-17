```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens. I attacked every bound, guarantee and claim about existing code in the decision record, D1 to D9 and Constraints.

**Claims attacked**
- D1, the detached key: a map call never joins a plain call's run. **Held.** `testRequest.words` feeds `detachKeyDir`'s hash, so adding `--map` to the words separates the keys.
- D1, the refusal list covers every flag that changes the call. **Held.** `lym gate test --help` shows `--tags` is already refused, and `--wait` and `--main-packages` are handled.
- D3, no test the run measures reads `LYM_PREBUILT_LYM`. **Held.**
  - `lymbin.prebuilt` is the only reader.
  - Tier0 files are untagged, and `ScanSpawns` gives untagged files an empty verdict; Test Tier Purity bans `lymbin.` in them.
  - `spawnRef` marks a direct `internal/testkit/lymbin` reference.
  - No kit wraps lymbin: only `_test.go` files import it.
  - `testmappackage.go:139` skips both `OutOfProcess` and `Unresolved` tests.
- D3, a tier's env carries no stale prebuilt path. **Held.** `gatecli/doc.go:113` drops an inherited `PrebuiltLymEnv`.
- D4, the alone-reruns stay within the side-by-side count. **Held.** `runBatch` (`testmappackage.go:351-363`) reruns only after the whole batch returns, one test at a time.
- D4, the tier's `memory_mb` bounds the test processes together. **Broken** on a host whose memory cap does not cover a process tree (F2).
- D4, the guard kills a `.test` tree past its limit and the run goes on. **Held.** `memoryGuard.run` kills the test binary's tree, and a kill is recorded as a failed outcome.
- D2, the one-ahead build is cancelled and waited for. **Held**, as stated.
- D6, a map is written only when its files are clean and HEAD has not moved. **Broken twice:** by the listing-failure exemption (F1) and by ignored files (F3).
- D6, its stated Bound on an edit reverted between checks. **Held**, as stated.
- D7, the slot hold is "the larger of tier 0's and tier 1's slot asks". **Held in meaning.** The Bound reads `slots: all` as every slot, but `Tier.Slots` encodes `all` as 0, so a plain integer max picks the wrong value (F5).
- D7, no green record and no run record are written. **Held** by construction.
- D8, last write wins and the map always comes from one measurement, never a mix. **Broken** by D7's keep-old-entries write on a failed build (F4).
- D8, old `.build.lock` files are inert. **Held.** Their only users are in `testmap.go`, `testmapplan.go` and `testmapaudit.go`, which D9 removes.
- Constraints, the `-trimpath` build-cache fact. Not attacked: it rests on the study report, which is outside my context.

**Findings**

F1 — MEDIUM, design, D6 (`decision-record.md:117`)
- A failed linked-package listing writes the map with no check at all.
- Counterexample: an uncommitted file in package P imports a package that does not exist. `go list` fails, and P's map records a failed build under HEAD's stamp, although P builds clean at HEAD.
- `lym gate redundant` then refuses P as "recording only a failed build" (D9) for a failure HEAD does not have. The same write also goes through after HEAD has moved.
- Neither check needs the linked set: HEAD can be compared, and the package's own files are known from `readPackageTree` (`internal/gatecli/testmaptree.go:27`). The exemption is broader than its stated reason.
- This contradicts D6's own title.

F2 — LOW, consistency, D4 (`decision-record.md:87-88`)
- "The tier's cap bounds the run's test processes together" does not hold on a host where `proc.MemoryCapCoversTree()` is false.
- There, `clampParallelToMemory` returns 1 (`internal/gatecli/redundantmutants.go:541-543`), and the per-test cap limits each process alone. One test that starts children can hold several times `redundancy_test_memory_mb`, which passes the tier's `memory_mb`.
- The tier's own scope is equally weak on such a host (`template.yaml:15`), but the record states the bound without that condition.

F3 — LOW, design, D6 (`decision-record.md:114`)
- The check sees only tracked files and `git status` entries.
- `readPackageTree` hashes `CodeTreeAt(rev)` (`testmaptree.go:28`), and `dirtyOwnedFile` reads `fabric.Status()` (`testmaptree.go:80`). Neither sees a gitignored file.
- Counterexample: a gitignored `.go` file in a package directory is compiled into the coverage binary. Its blocks and outcomes then enter the map under HEAD's hashes, unseen.
- D6's Bound does not name this case.

F4 — LOW, consistency, D8 (`decision-record.md:149,153`) against D7 (`:136`)
- "Each write replaces the map at once … never a mix" fails when one of two concurrent runs has a failed build.
- D7 keeps the old entries on a failed build, so that write merges with the map it reads.
- Interleaving: runs A and B start over P from one worktree. B writes a fresh map. A's coverage build then times out under the doubled load, and A writes B's entries plus A's failed-build record, or older entries if A read the map before B's write.
- The final map mixes two runs, against D8's rationale.

F5 — NIT, consistency, D7 (`decision-record.md:130`)
- "The larger of tier 0's and tier 1's slot asks" is ambiguous against the config encoding: `Tier.Slots` stores `slots: all` as 0 (`internal/gateslot/tiers.go:42-43`).
- A plain integer max of tier0 `slots: 1` and tier1 `slots: all` gives 1, not every slot.
- D7's Bound shows the intended meaning, but the decision text doesn't state that `all` ranks highest.

**Tool calls**
1. `testmaptree.go` file listing and dirty check (grep): needed to see whether D6's check sees ignored files.
2. `.build.lock` users and map-directory `ReadDir` (grep): needed to test D8's "inert" claim.
3. The alone-rerun ordering in `testmappackage.go` and the memory guard's sampling root (grep): needed to test D4's count against D5's rerun.
4. `testmaptree.go:20-100`, `testmap.go:430-450` and `runBatch` (sed): the bodies behind calls 1 to 3.
5. Detached-key functions in `detachfront.go` (grep): needed to test D1's keying claim.
6. `detachfront.go` `words`, `detachKeyDir` and `longWait` (sed): the bodies behind call 5.
7. `redundancy_test_memory_mb` validation and the `proc` memory helpers (grep): needed to test D4's quotient for a zero or unset cap. This is a seventh call, one over the six-call limit.
