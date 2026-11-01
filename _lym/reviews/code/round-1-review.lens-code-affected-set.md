```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set — per change, who reads or asserts it outside the touched lines.

**Per change**

1. **`driverStartGrace` and the grace branch in `noticeCondition`** (`internal/dobbyengine/runnotice.go:251`):
   - Callers: `runNotice` is the only caller.
   - Tests in `internal/dobbyengine/runnotice_test.go`: the three rows that expect a dead-driver notice (`:108`, `:118`, and the `OncePerEpisode` test at `:363`) and the retry test at `:490` now advance past the grace. Covered.
   - Docs: `internal/dobbyengine/doc.go:127` is updated. The `doc.go:133` sentence on driver-dead key lifetime still holds, because the grace changes when the key is first made, not when it ends. Covered.
   - Stale doc: `internal/dobbyengine/doc.go:136` says a machine that takes a run over dates the child's since from its own first sight, "so a driver-dead notice the other machine sent can go once more." The grace now counts from that same since, so a takeover also delays the notice by up to a minute. The sentence is not wrong, but it omits this effect. See the LOW finding.
   - `internal/battenshed/notice.go` and `internal/battenshed/doc.go:90,93` name the driver-dead notice, but only to say that batten sends none for a running child. Unaffected.
   - Other tests that drive `noticeCondition` with a dead driver: `runnotice_test.go` only. `internal/beamcli/dobbydeps_integration_test.go:75` and the `internal/battencli` tests set `DriverAlive` only as a nil check or as a batten inner-run seam, so no wiring test asserts a dead-driver notice. Covered.

2. **`childDriverLockHeld` now also checks `loomengine.LoomBootstrapLock`** (`internal/battencli/dobbynotice.go:131`):
   - Callers: `noticeDriverAlive` only. The test row is added in `dobbynotice_test.go`. Covered.
   - `internal/battencli/wire.go` already imports `loomengine`, so the new import adds no layering change. `internal/battencli/drive.go` imports it too.
   - **Stale doc:** `internal/battenshed/doc.go:119` still says "one with neither lock and no strand is noticed as driver-dead". That sentence describes the driver-dead notice and its locks, and the notice now also needs a held bootstrap lock to be absent and a minute of awake time to have passed. It is MEDIUM-adjacent, but it only describes the notice and does not govern behavior, so I rate it LOW.
   - **Stale doc:** `internal/battenshed/doc.go:161` says "with neither lock held and no strand, is noticed as driver-dead once the owner pause lifts and an owned-here verdict follows". It has the same gap, and the grace also applies after the pause lifts. The grace counts from the since of the running episode and excludes paused intervals, so this sentence is still roughly right.

3. **`DriverAlive` seam doc** (`runnotice.go:102`) is updated. Covered.

4. **Commit message and `Pattern-Departure:` trailer**: the commit has no trailer.
   - It needs none. The diff touches no PATTERN entry. The `loomengine` import adds no `PATTERN-leaf-packages` violation, and `battencli` is not a leaf.
   - The `PATTERN-daemon-wakeups`, `PATTERN-run-process-owner` and `PATTERN-test-speed` entries are not newly hit. The new tests use the injected clock.
   - No PATTERN declarations are listed in the directive, so nothing is owed on that point.

**Findings**

- **LOW, consistency** — `internal/battenshed/doc.go:119`: the paragraph says a running Go child with neither lock and no strand is noticed as driver-dead. That omits the loom bootstrap lock, which now counts as live, and the one-minute awake-time grace. Evidence: `internal/battenshed/doc.go:119` against `internal/battencli/dobbynotice.go:131` and `internal/dobbyengine/runnotice.go:251`. Fix: name the bootstrap lock and the grace in that sentence, or drop the lock list from it.
- **LOW, consistency** — `internal/dobbyengine/doc.go:136`: the takeover "Bound" sentence omits that a machine dating the child's since from its own first sight also restarts the driver-start grace. Evidence: `internal/dobbyengine/doc.go:136` against `runnotice.go:251`. Fix: add a clause saying the notice then also waits the grace from that first sight.

Everything else I looked at is covered.

**Tool calls**
1. grep for the driver-dead wording across the repo: to find every doc, test or fixture that describes or asserts the condition.
2. grep for `DriverAlive`, `childDriverLockHeld` and `noticeDriverAlive`: to find the callers and readers of the changed seam.
3. grep in the beamcli and battencli tests, plus `sed` of `battenshed/doc.go:115-121,159-162`: to check whether any wiring test asserts the notice, and to read the two stale-doc candidates.
