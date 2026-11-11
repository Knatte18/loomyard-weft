```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the code the fixer changed, through the fix-code-correctness lens.

**LOW: `HubEnvDigest` assertion cannot fail**
- Changed text: the check `stamp.TiersDigest == "" || stamp.HubEnvDigest == ""` in `TheMapRecordsLinkedFunctionsAndFilesTheSpawnVerdictAndAMarker`.
- Evidence: `environmentStamp` sets `HubEnvDigest = ContentHash([]byte(strings.Join(r.hubEnv.Entries(), "\n")))`, and the fixture hub's `env.yaml` is empty. The hash of an empty string is still non-empty, so a regression in how the digest is built passes this check.
- The review asked only for "non-empty", so the fix matches it. `TiersDigest` has real coverage, because the new subtest asserts it is unchanged by the parallel count and different after a command change.

No other findings. I traced each changed branch, and each added test fails with its defect in place:
- **`mapPackage` returns:** every path returns `(bool, error)` correctly, and only the before-build dirty skip returns true.
  - The `file, err :=` after `own, err :=` redeclares nothing, and the later `file, err =` reuses both.
  - `mapPackages` is the only caller, and its `skippedBeforeBuild, err :=` is legal because `skippedBeforeBuild` is new.
- **F1:** with a cap of 1 and `caller`, `follower`, `linked/inner` named, the old behaviour leaves `built` at 1 after `caller`. `follower` and `inner` would then be unbuilt, so the test's expected `Built: [linked/inner]` and `Unbuilt: 0` fail.
- **F5:** a status read reused from before the walk would not see `scratch.txt`. `follower` would then build and be skipped with the "after its tests" suffix, which does not match the expected cause.
- **F2:** if the HEAD-moved check covered only the package's own tree, the commit to `linked/linked.go` would go unseen and a map would be written. The test would then fail.
- **F4:** a dropped `go.mod` branch would put `modnest/go.mod` into `Linked`. A wrong `buildableDir` would drop `testdata/only.go`, `_hidden/hidden.go` or `.dotdir/dot.go` from it. Either way the exact-map diff fails.
- **F6:**
  - The `go list` failure fails inside `LinkedPackages` and reaches `writeFailed`, so the map carries the marker.
  - The garbled non-test file breaks `ScanSpawns` but not `ScanTestFuncs`, so the failure message names the spawn scan.
  - The `go env` failure surfaces on the first named package. `linked` has no tests, so `write` calls `writeMap`, and `environmentStamp` fails there.
  - `TestCallerUnresolved` gets `Unresolved` because `call := func(){}; call()` is a local function value.
- **F7:** a tiers digest taken after the `{parallel}` substitution would change between parallel 1 and 2, so `again != first` would catch it.

Tool calls: none; my context already held every changed file and the gate output.
