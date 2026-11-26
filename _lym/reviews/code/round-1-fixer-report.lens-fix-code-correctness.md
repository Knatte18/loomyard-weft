```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the fix's changed code (lens fix-code-correctness).

**1. MEDIUM: nothing fails if the beam dobby's `Runs` seam ignores the injected start.**
- Changed text: `internal/beamcli/dobbydeps.go`, `hubDobbyDepsStarting`, `battencli.DobbyRuns(..., startPusher)` in `Runs`, and the new test `TestHubDobbyDeps_PrimeTickStartsThePrimesPusherAndABeamsTickNone`.
- The test calls only `primeDeps.Tick` and checks that `beamDeps.Tick` is nil. It never calls `deps.Runs()`.
- If `Runs` passed `runpusher.StartIfAhead` or a no-op instead of `startPusher`, every test would pass.
- F3 asked for a tick to be shown calling the start "once for the prime and once for each owned pair". The pair half is asserted only on `DobbyRuns` called directly.

**2. LOW: the production wiring of the resume and read starts is still unpinned.**
- Changed text:
  - `internal/battencli/wire.go`: `c.startPusher = func() { startRunPushers(..., runpusher.Start) }`
  - `internal/loomcli/wiring.go`: `c.startPusher = func() { startRunPusher(...) }`
  - the `hubDobbyDeps` wrapper and `DobbyViewReader.Read`
- `TestStartRunPushers_...` and `TestStartRunPusher_...` call the extracted helpers directly. Deleting either `c.startPusher` assignment, or its call, leaves them green.
- F3's premise was that these closures "would pass every test if their calls were deleted", and that still holds one level up. Its `fix` asked to assert through the seam "that the closure passes" the targets.
- The fixer-report should say this rather than claim the closures are covered.

**3. LOW (mainly the consistency lens's area): the `DobbyRuns` doc line is wrong about the condition.**
- Changed text: `internal/battencli/dobbyreads.go`, "first restarts its pair's pusher through startPusher when the pair holds commits origin lacks."
- `DobbyRuns` calls `startPusher` for every owned, present pair. The ahead condition lives only in the function passed in.
- The line is also longer than 120 characters.

No wrong results or broken callers found:
- **Push bound:** three pushes, then Stuck. `PushPairAnchored` returns an owed error for a side still unpushed, which `PushUntilLanded` retries, so a deferred push does not use up the three rounds.
- **Pushes off:** `Push` returns nil and `Level` returns true, so the row reaches the removal.
- **Defect-sensitivity:** the bound, skip, error-mapping and F8 tests would each fail with their defect in place, by reading. I did not run any of them against the old code.
- **F7:** the test drives the pruned-ref push. A skipped versus run rewind check is not observable in it, although the comment mentions "no rewind check".

Tool calls (1 of 6):
- Grepped `PushRun` and `IsPushOwed` and read their surroundings in `pairteardown` and `fabricengine`: my context did not show whether a deferred push returns nil, which decides whether the three-round bound can trip on a transient lock.
