# Card 1 — OpenSpec carries a permission mode

**Edit:**
- `internal/shuttleengine#OpenSpec`
- `internal/shuttleengine#SessionOpener`
- `internal/shuttleengine#Runner.Open`
- `internal/shuttleengine#TestRunner_Open`
- `internal/shuttleengine/doc.go`

**Uses:**
- `internal/testkit/logcapture#CaptureVerbose`
- `internal/shuttleengine#openerFakeEngine`
- `internal/shuttleengine#fakeReed`
- `internal/shuttleengine#newFixture`

**Intent:** Give `OpenSpec` a `PermissionMode string` field (D2), documented as caller-owned engine vocabulary exactly as `Spec.PermissionMode` is in `internal/shuttleengine/spec.go`: empty keeps the engine's interactive default, `Runner.Open` and the CLI pass it uninspected, and the engine is the sole validator and realizer.
The `OpenSpec` type comment, which lists what the spec describes, names the permission mode too.
The `SessionOpener` doc comment says an implementing engine honours `spec.PermissionMode` or refuses it from `OpenLaunch`, so the refusal lands before `Runner.Open` mints the marker token, creates the marker directory or adds a strand.

`Runner.Open` passes `spec` to `OpenLaunch` unchanged, as today, and its existing `"shuttle: opened session"` `logger.Info` line gains a `"permissionMode", spec.PermissionMode` pair, so a bypass open is traceable afterwards (D5, `PATTERN-spawn-observability`).
Its existing refusals (empty role, unknown engine, covering pause, engine with no `SessionOpener`) stay ahead of the `OpenLaunch` call, so they keep reporting ahead of a mode refusal (D3).

The `# Opened sessions` paragraph of `internal/shuttleengine/doc.go` gains one sentence: `OpenSpec.PermissionMode` is engine vocabulary carried blind to `OpenLaunch`, and the engine realizes it or refuses it before anything is created.
The paragraph that names `Spec.PermissionMode`, `Spec.AllowAgentTool` and `Spec.ResumeSessionID` as caller-owned engine vocabulary adds that `OpenSpec.PermissionMode` is the same vocabulary, carried blind by `Runner.Open`, with empty keeping the engine's interactive default.

Tests, all in `TestRunner_Open`:
- The refusal loop, after its `AddStrandCalls` check, asserts that `filepath.Join(fx.Anchor, lyxdirs.DotLyxDirName, openMarkerDirName)` does not exist, so every refused open, the "a failed open launch is returned" row standing for a mode refusal included, is pinned to create no marker directory (AC4).
- The "a live pause covering the worktree is refused" row's engine becomes `&openerFakeEngine{openErr: errors.New("provider down")}` while it keeps wanting "re-run lyx shuttle open", so the pause refusal is pinned to report ahead of an engine refusal (D3).
- The "adds an own-window strand by role" subtest opens with `PermissionMode: "bypass"` added to its spec, captures logs with `logcapture.CaptureVerbose` before the open, and asserts that the engine's recorded `openSpecs` holds exactly one spec whose `PermissionMode` is `"bypass"` and that the captured output holds `permissionMode=bypass`.
  Because it captures the process-global logger, that subtest drops `t.Parallel` and says so in a comment, as `TestReadAgentWaits`' log subtest does, and `TestRunner_Open` drops its own `t.Parallel` so no parallel test of the package swaps the logger output during the capture; its other subtests stay parallel among themselves.

**ImpactSummary:** One new struct field and one log attribute in `internal/shuttleengine`; the two `SessionOpener` test fakes (`openerFakeEngine`, `dispatchEngine`) ignore the field and need no change, and no caller of `OpenSpec` sets it yet.

**Commit:** 1: OpenSpec carries a permission mode to the engine and the open log line
