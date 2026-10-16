# Card 1 — reed-styled-capture

**Edit:**
- `internal/reedengine#Engine.CapturePane`
- `internal/reedengine/doc.go`
- `internal/shuttleengine#ReedOps`
- `internal/testkit/shuttlefake#Reed`
- `//cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `plan:internal/reedengine#Engine.CapturePaneStyled` -> `func (e *Engine) CapturePaneStyled(guid string) (string, error)`
- `plan:internal/reedengine#Engine.capturePane` -> `func (e *Engine) capturePane(guid string, styled bool) (string, error)`
- `plan:internal/reedengine#TestCapturePaneArgv` -> `func TestCapturePaneArgv(t *testing.T)`
- `plan:internal/shuttleengine#fakeReed.CapturePaneStyled` -> `func (m *fakeReed) CapturePaneStyled(guid string) (string, error)`
- `plan:internal/testkit/shuttlefake#Reed.CapturePaneStyled` -> `func (r *Reed) CapturePaneStyled(guid string) (string, error)`

**Uses:**
- `internal/reedengine#newTestEngine`
- `internal/reedengine#installIfAbsentTmux`
- `internal/reedengine#fakeTmux.ArgvFor`
- `internal/reedengine#fakeTmux.answer`
- `internal/reedengine#resolveLivePaneID`

**Intent:** Decision D1, reed half: reed gains a styled pane capture, the screen with its SGR attributes, as a provider-blind read beside `CapturePane`.

`Engine.CapturePane` and the new `Engine.CapturePaneStyled` both go through one unexported `Engine.capturePane(guid, styled)`, which keeps today's lock, session check, state load and `resolvePaneInThisSessionLocked` lookup, and runs `capture-pane -p -t <pane>` for the plain read and `capture-pane -e -p -t <pane>` for the styled one.
The styled read returns tmux's output verbatim; reed interprets none of it (`PATTERN-shuttle-provider-seam`), and it keeps `CapturePane`'s read-only discipline.
`capture-pane` is already in `requiredSubcommands`, so the probe is unchanged.
`io.go`'s file header stays as it is, since it describes the capture family `CapturePane` heads, and `internal/reedengine/doc.go` gains one sentence saying reed offers the styled capture for a caller that reads cell styles and never interprets them.

`shuttleengine.ReedOps` gains the interface method `CapturePaneStyled(guid string) (string, error)`, so `*reedengine.Engine` still satisfies it as-is.
Its two other implementers gain the method: shuttle's `fakeReed.CapturePaneStyled` answers exactly as `fakeReed.CapturePane` does, from the same `CaptureQueue`, `CaptureErr` and `CaptureErrQueue`, logging `CapturePaneStyled` in `CallLog`; `shuttlefake.Reed` gains a `CapturePaneStyledFn` field and its `CapturePaneStyled` records the guid in `CaptureCalls` and answers `CapturePaneStyledFn`, else an empty capture, as `CapturePane` answers its own function.
No production caller uses the styled read yet; card 3 does.

`TestCapturePaneArgv` is a new table test in `io_test.go` over `newTestEngine` and `installIfAbsentTmux`, with a strand bound to `%7` saved in the state as `TestPasteSubmit` does: a row per read asserts the `capture-pane` argv (`-p -t %7` plain, `-e -p -t %7` styled) and that the method returns the fake's `capture-pane` answer unchanged, including SGR bytes for the styled row; a row per read where the fake's `capture-pane` answers an error returns an error wrapping it, so a failed styled capture reaches its caller.
The unknown, hidden and unbound refusals both reads share stay pinned by `TestResolveLivePaneID`.
No existing test pins `CapturePane`'s argv, so this is a new top-level test, and `cmd/lyx/testdata/test-budget.yaml`'s `internal/reedengine` row rises from 269 to 270.

**ImpactSummary:** Adds one reed read and one ReedOps method; the only ReedOps implementers are reed's Engine, shuttle's fakeReed and shuttlefake.Reed, all updated here.
