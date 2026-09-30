# Card 4 — Sink arm-state query

**Edit:**
- `internal/logger/sink.go`
- `internal/logger/sink_test.go`
- `internal/logger/sink_callsite_integration_test.go`

**Create:**
- `internal/logger#SinkArmState`
- `internal/logger#CurrentSinkArmState`

**Intent:**
Give the CLI layer the query decision record D3 and D4 need: whether the durable sink armed in the current generation, whether it was redirected, where it writes, and which lyx anchor it resolved from.

In `sink.go`:

- Add `SinkArmState` with fields `Armed bool` (the sink came up in the current generation), `Redirected bool` (a `SetDurableSinkDir*` override was in force), `Dir string` (the sink directory), and `AnchorPath string` (the lyx worktree anchor the cwd-anchored resolution used; empty when redirected).
- Make `resolveSinkDirLocked` also return the anchor path (`layout.AnchorPath()`), empty for an override, so no new cwd resolution is added.
- Record the sink directory and anchor path in two new package vars under `sinkMu` when `armDurableSinkLocked` succeeds, and clear them in `resetDurableSinkLocked`.
- Add `CurrentSinkArmState()`, which takes `sinkMu` and reports `Armed` as `sinkArmed && sinkOK`, `Redirected` as `sinkDirOverride != ""`, and the recorded directory and anchor.
  It never arms the sink, creates anything, or logs.
- Update `TraceDir` for the new `resolveSinkDirLocked` return value.

In `sink_test.go`, add tests.
Each case starts with `SetDurableSinkDir(dir)` on its own `t.TempDir()`, per the file's existing convention:
under `go test` without `LYX_TRACE=1`, `resolveSinkDirLocked` refuses the cwd-anchored path, so without the redirect no case can arm.

- Right after the redirect, before any record, `CurrentSinkArmState()` reports `Armed: false`, and calling it leaves `dir` empty.
- After one `Info` record, it reports `Armed: true`, `Redirected: true`, `Dir: dir`, `AnchorPath: ""`.
- After `NotifyExit(1)` with no prior record, it reports `Armed: true`, `Redirected: true`, `Dir: dir`.
- After an arm followed by a redirect to a second `t.TempDir()`, it reports `Armed: false` again until the next arm.

In `sink_callsite_integration_test.go`, extend `TestDurableSink_CwdFallbackArmsInALyxOwnedWorktree` (or add a sibling test using `armFallbackSinkFrom`) to assert the cwd-anchored arm reports `Redirected: false`, `Dir` equal to `<repo>/.lyx/logs`, and `AnchorPath` equal to `repo`.

**ImpactSummary:** `resolveSinkDirLocked` gains a return value; its callers `armDurableSinkLocked` and `TraceDir` are both in `sink.go`, and neither's observable behaviour changes.

**Commit:** 4: logger: expose the sink's arm state for the exit sweep
