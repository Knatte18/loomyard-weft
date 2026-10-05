# Card 7 — notice-queue

**Edit:**
- `internal/orchengine/state.go`
- `internal/orchengine/watcher.go`
- `internal/orchengine/watcher_test.go`
- `internal/orchengine/watchloop.go`
- `internal/orchengine/watchloop_test.go`
- `internal/orchengine/doc.go`
- `internal/orchcli/paths.go`
- `internal/orchcli/doc.go`
- `PATTERN.md`

**Create:**
- `internal/orchengine/notice.go`
- `internal/orchengine/notice_test.go`

**Intent:**
Give the orch a notice queue that only the orch watcher types from (D13), the Go side of `orch-no-watcher`.

Paths: `orchengine.Paths` gains `NoticesDir`, and `internal/orchcli/paths.go` joins it as `notices` under the prime's `.lyx/orch/` through `lyxdirs` (PATTERN-lyxdirs-single-declarer, PATTERN-durable-vs-ephemeral-state).
`paths.go` also exports `PrimePaths(location *lyxcwd.Location) orchengine.Paths`, returning what `orchPaths` returns, so another module's CLI wiring can reach the queue and the orch state without re-deriving either (PATTERN-told-geometry: the engines still take the paths from their callers).

`notice.go` in `orchengine` owns the queue, all on told paths:
- `QueueNotice(p Paths, line string, now time.Time) (queued bool, err error)` loads the orch state and, only when it records a strand, writes the line as one file under `NoticesDir`, atomically (temp file plus rename), named so names sort by arrival (UTC time to the nanosecond plus a random suffix), so concurrent writers never share a file.
  A notice is typed through `Send` as one line, so it refuses a line containing a newline or beginning with `/` with an error and writes nothing.
  With no strand recorded it logs the notice through `internal/logger` and returns false.
  After writing, it enforces a cap of 50 notices by removing the oldest beyond it and logging each drop; concurrent appenders may overshoot the cap transiently by at most one notice each.
- Listing the queue oldest first, removing one notice, and dropping the whole queue.

Watcher: in phase idle, after `tickIdle` has not started a cycle on this tick, the watcher delivers the oldest queued notice when the idle probe passes on the same tick: `Send` it as a typed turn, then remove its file.
One notice per tick; a non-idle probe, a running cycle (any non-idle phase) or a watcher that is down leaves the queue untouched.
Delivery is at least once: a watcher that dies between typing and removing types the notice again after restart.
`Watcher.Run` in `watchloop.go` drops the queue at start when the orch state records no strand.

Bound: a notice can be delayed by a busy session, a too-short pane or a cycle, but is never typed over a draft or a running turn; the queue never durably grows past its cap.

Tests in `notice_test.go`: no strand recorded logs only and writes nothing; a multi-line or `/`-leading line is refused; files sort by arrival; the cap of 50 drops the oldest; listing is oldest first.
In `watcher_test.go`: delivery is oldest first, one per tick, only in phase idle on an idle-probe pass, and removes the delivered file; a non-idle probe or a non-idle phase leaves the queue untouched; a tick that starts a cycle delivers nothing.
In `watchloop_test.go`: `Run` drops the queue when no strand is recorded.

`PATTERN.md` gains, under "Agents and prompts", `PATTERN-orch-pane-single-writer` — Typing into the orch session from Go: only the orch watcher does it, idle-gated; another module queues a notice through `orchengine` instead.
Docs: `internal/orchengine/doc.go`'s `.lyx/orch/` layout section names `notices/`, and a notices section states the delivery rules above; `internal/orchcli/doc.go` names `PrimePaths` as the one accessor other modules use.

**ImpactSummary:** Adds a notice queue and its delivery step to the orch watcher, one Paths field and one exported orchcli accessor, plus a new PATTERN entry; existing cycle behaviour is unchanged.
