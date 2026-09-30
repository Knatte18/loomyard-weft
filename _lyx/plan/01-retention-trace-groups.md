# Card 1 — Sweep by trace group and last activity

**Edit:**
- `internal/logger/retention.go`
- `internal/logger/sink.go`
- `internal/logger/logger.go`
- `internal/logger/retention_test.go`

**Create:**
- `internal/logger#RetentionBounds`
- `internal/logger#DefaultRetentionBounds`

**Intent:**
Rewrite the retention sweep so the count bound counts traces rather than files, and both bounds rank by last activity rather than filename timestamp (decision record D1, D2).

In `retention.go`:

- Add `RetentionBounds` with two fields, `Count int` (traces to keep) and `MaxAge time.Duration` (age bound), and `DefaultRetentionBounds()` returning `Count: 200`, `MaxAge: 14 * 24 * time.Hour`.
  These are the compiled-in defaults every process uses until the exit path passes configured bounds; card 2 pins them equal to the `logger.yaml` template.
- Remove `retentionAgeBound`, `retentionCountBound`, `retentionCandidate`, and `minInt`.
- Change `traceFilePattern` to capture the trace id as its own group; the filename grammar itself does not change.
- Change `Sweep` to `Sweep(dir string, bounds RetentionBounds) error`, implemented over an unexported helper that also takes `now time.Time` and a stat function (`func(string) (fs.FileInfo, error)`), so a test can stub a failing stat; `Sweep` passes `time.Now()` and `os.Stat`.
- Group every matching, non-directory entry by trace id.
  A group's activity time is the newest mtime among its files; a file whose stat fails contributes its filename timestamp instead, and the failure neither panics nor aborts the sweep.
  A group is live when any of its files carries the sweeping process's own pid or a pid `proc.IsAlive` reports alive.
- Live groups are never deleted and do not consume count budget.
- A non-live group whose activity time is older than `now - bounds.MaxAge` has every file deleted.
- The remaining non-live groups are ranked by activity time, newest first; the first `bounds.Count` survive and every file of each later group is deleted.
- Non-matching names and subdirectories stay untouched, delete failures stay silently tolerated, and an absent or empty directory returns nil.
- Rewrite the file header and `Sweep`'s doc comment to state these rules; drop the "ranked by filename timestamp, never mtime" line.

In `sink.go`'s `armDurableSinkLocked`, change the call to `Sweep(dir, DefaultRetentionBounds())`.
The sweep stays at arm for this card; card 5 moves it to process exit.

In `logger.go`, rewrite the "The durable sink and its retention" section's sweep paragraph to the new rules: grouping by trace id, activity time as the newest mtime, the live-group exemption, and the 200-trace / 14-day defaults.

In `retention_test.go`, update every existing test to the group and activity-time semantics rather than deleting any.
`writeTraceTestFile` sets each file's mtime to its filename timestamp with `os.Chtimes`, so ranking is deterministic, and tests pass small explicit `RetentionBounds` instead of writing 200 files.
Add tests for:

- Several trace ids, some with multiple files: the newest `Count` groups by activity survive whole, and every file of each older group is deleted.
- A group with one live-pid file (the test process's own pid) keeps all its files regardless of age and does not count toward `Count`.
- The Webster-step scenario: a group whose filename timestamp is the oldest but whose mtime is the newest survives a count-bound sweep that deletes `Count`+ groups with later filename timestamps and older mtimes.
- A non-live group whose newest mtime is older than `MaxAge` is deleted even when `Count` would keep it; a live group older than `MaxAge` is kept.
- A stubbed stat that fails for one file: that file ranks by its filename timestamp, and the sweep still returns nil and processes the rest.
- Non-matching filenames and subdirectories untouched; absent and empty directories return nil.

**ImpactSummary:** `Sweep`'s signature changes; its only production caller is `armDurableSinkLocked`, updated here, and `retention_test.go` is its only other caller.

**Commit:** 1: logger: sweep trace files by trace group and last activity
