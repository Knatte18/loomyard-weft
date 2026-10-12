# Card 12 — the provider seam can open a named session with no prompt

**Create:**
- `plan:internal/shuttleengine#OpenSpec` -> `type OpenSpec struct`
- `plan:internal/shuttleengine#SessionOpener` -> `type SessionOpener interface`
- `plan:internal/shuttleengine/claudeengine#Claude.OpenLaunch` -> `func (c *Claude) OpenLaunch(spec shuttleengine.OpenSpec, cfg shuttleengine.Config) (shuttleengine.Launch, error)`
- `plan:internal/shuttleengine/claudeengine#buildOpenCmd` -> `func buildOpenCmd(sh shell.Shell, bin, sessionID, model, effort string, resume bool) string`
- `plan:internal/shuttleengine/claudeengine#TestOpenLaunch` -> `func TestOpenLaunch(t *testing.T)`

**Edit:**
- `internal/shuttleengine/claudeengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shuttleengine#Launch`
- `internal/shuttleengine/claudeengine#validateEffort`
- `internal/shuttleengine/claudeengine#resolveModelID`
- `internal/shuttleengine/claudeengine#newSessionID`
- `internal/shuttleengine/claudeengine#claudeBinary`
- `plan:internal/shell#posixShell.OrElse`

**Intent:**
D12's provider specifics, kept in `claudeengine` (`PATTERN-shuttle-provider-seam`): the launch line of a named interactive session with no prompt, the session name flag, and the resume of a session that may hold no turns.

In `internal/shuttleengine/engine.go`:

- `OpenSpec` carries `Role string` (the strand role, from the caller's flag), `Model`, `Effort` and `Version string` (the resolved model choice, each empty for the provider default) and `Segment segmentcolor.Segment`.
- `SessionOpener` is an optional capability beside `Engine`, like `SessionResumer`: `OpenLaunch(spec OpenSpec, cfg Config) (Launch, error)` returns the launch and resume lines and the session id of a session that runs with no prompt, no run directory and no run artifacts.
  Its doc comment says the session runs under the caller's own worktree settings, takes no turn, and is opened by `Runner.Open` (card 13).

In `internal/shuttleengine/claudeengine/command.go`:

- `buildOpenCmd(sh, bin, sessionID, model, effort, resume)` composes `claude --session-id <id>` (or `--resume <id>` when `resume`), then `--name` as a reference to `LYX_STRAND_NAME` (reed exports it before the line runs), then `--model` and `--effort` only when non-empty, all quoted through `sh`.
  It passes no prompt, no `--settings` (the session takes the worktree's own settings), no permission flag (an interactive session prompts as the operator's would) and no environment assignment, so each line is one pipeline that `OrElse` may join.
- `Claude.OpenLaunch(spec, cfg)` validates the effort and resolves `spec.Model` and `spec.Version` into a model id before anything else, mints a session id, and returns `Launch{Cmd: fresh, ResumeCmd: sh.OrElse(resumeLine, fresh), SessionID: id}`.
  The resume line falls back to a fresh start under the same id because Claude Code keeps no transcript for a session that never took a turn, so `--resume` fails there and the session must start again under the same name and id.
  `RecordsSessionStarts` stays false, since nothing waits on an events file.
- A compile-time assertion that `*Claude` satisfies `shuttleengine.SessionOpener` sits beside the method.

`internal/shuttleengine/claudeengine/doc.go` gains a paragraph on the open launch: no prompt, the worktree's settings, the name reference, and the resume that falls back to a fresh start for a session with no turns.

`TestOpenLaunch` in `command_test.go`, a table over both dialects:

- `fresh line`: model `opus`, effort `high`: `Cmd` starts with the quoted binary and holds `--session-id '<id>'`, the strand-name reference, `--model 'opus'` and `--effort 'high'`, and holds neither `--settings`, `--dangerously-skip-permissions` nor a prompt pointer.
- `resume falls back to a fresh start`: `ResumeCmd` equals the dialect's `OrElse` of the `--resume` line and `Cmd`.
- `no model or effort adds neither flag`.
- `an invalid effort is refused` and `a version without a model is refused`, each before any session id is returned.

`cmd/lyx/testdata/test-budget.yaml`: raise `internal/shuttleengine/claudeengine` by one.

**ImpactSummary:** Adds an optional engine capability and its Claude implementation; no existing caller or fake engine changes, since nothing requires the capability before card 13.

**Commit:** 12: The provider seam opens a named session with no prompt
