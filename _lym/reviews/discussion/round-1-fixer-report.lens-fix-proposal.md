```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I tested the review's proposals that the fixer carried out as written, looking for concrete cases where each one is wrong.

**MEDIUM: F12 together with D8's start rule creates a restart loop at a fixed HEAD.**
- Changed text: D1, "that map counts as queued work in the exit record (D8)". D8, "Queued work is the work left when the pass ended, a map the plan skipped for a held lock included (D1)".
- Case: a hand call (D9) holds package P's build lock through a whole rebuild that runs for minutes. On every dobby tick, the dobby starts a pass because the last exit record names queued work. That pass plans every map through `CodeFileAt`, skips P because its lock is held, and ends with P queued again. The cycle repeats each tick until the hand call releases the lock.
- Evidence: D8's carve-out bullet says the carve-outs exist "so neither a dirty prime nor a broken commit restarts passes in a loop". A held lock is a third transient state, and F12 adds it to the queue with no carve-out and no back-off.

**MEDIUM: F8's bound on a hand `--upkeep` pass does not hold.**
- Changed text: D8, "A hand caller may pass it too ... The build lock per package keeps it and any other pass off the same package, and the dobby starts no pass while a pass registered as `runproc.RoleTestMap` lives."
- Case: a hand `--upkeep` call is never registered, so the dobby can start a second pass beside it. Both passes write the exit record. Suppose the dobby's pass ends with an empty queue because the hand pass held the locks, and it writes last. Its record then claims no queued work, and work the hand pass left stalls until the next landing.
- Evidence: only `startTestMap` registers the refresh, through `runproc.Register(runProcDir, runproc.HubOwner, record)` in `internal/beamcli/dobbydeps.go` (≈ lines 617–620). A hand call takes no such path.

**MEDIUM: F5's lock release lets a yielded pass overwrite a hand call's map.**
- Changed text: D9, "releases the slot and the build lock of the package whose job it interrupted ... it takes that lock again with the slot". D9's Bound, "A hand call's wait for a build lock ends within one pass job".
- Case: the pass yields on P and releases P's lock. A hand call takes the lock, rebuilds P whole and writes P's map. The pass then retakes the lock and resumes its job. D10's "The job writes the map after each round" then writes the pass's pre-yield view of P's map, dropping or reverting the hand call's fresh entries. Nothing says the resumed job reads the map again, or that it ends, once it retakes the lock.
- Evidence: D10's write-per-round bullet together with the new release/retake text in D9.

**MEDIUM: F9's rule misses Go files under `testdata`.**
- Changed text: D2, "The package's own files other than Go files are every tracked file under its directory tree down to the next package directory, `testdata` included". The same rule is in D3, and the D2 criterion still says "an edit to a file under B's `testdata` is a miss".
- Case: Go fixture files under `testdata` are not "files other than Go files", and they are not package Go files compared by function either. An edit to one is therefore no miss, even though the tests load them as fixtures. That contradicts the D2 criterion.
- Evidence: `git ls-files` lists `internal/planglyph/testdata/glyphchain/callees/callees.go` and `local.go`.

Tool calls beyond my context:
- `internal/fabriccli/anchordeps.go` lines 235–258 and 471–500: the spare list and the body of `dobbyPassInFlight`, to test F7's mechanism.
- `dobbydeps.go` registration, the `EndDobbies` spare roles and `git ls-files` for `testdata/*.go`: who registers a pass (F8), and Go fixture files under `testdata` (F9).
- `grep` for `dobbyOwner` and `HubOwner`: whether `EndDobbies` reaches the hub-owner `RoleTestMap` record.
- `internal/fabriccli/anchordeps.go` lines 88–102 (the `dobbyOwner` closure): it confirmed that the prime maps to `HubOwner`, so F7's mechanism holds.
