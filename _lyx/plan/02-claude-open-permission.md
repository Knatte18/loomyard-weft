# Card 2 — Claude's open launch realizes the permission mode

**Edit:**
- `internal/shuttleengine/claudeengine#validatePermissionMode`
- `internal/shuttleengine/claudeengine#buildOpenCmd` -> `func buildOpenCmd(sh shell.Shell, bin, sessionID, model, effort string, skipPermissions, resume bool) string`
- `internal/shuttleengine/claudeengine#Claude.OpenLaunch`
- `internal/shuttleengine/claudeengine#TestOpenLaunch`
- `internal/shuttleengine/claudeengine/doc.go`

**Uses:**
- `internal/shuttleengine/claudeengine#engineWith`
- `internal/shuttleengine/claudeengine#templateConfig`

**Intent:** `Claude.OpenLaunch` resolves `spec.PermissionMode` through `validatePermissionMode(spec.PermissionMode, true)` right after its effort and model checks and before `newSessionID` (D3), so `bypass` resolves to skip, `prompt` and empty resolve to no skip, and any other value returns the existing `invalid permission mode` error naming the legal values, with no session id minted.
An opened session is always interactive, so the existing interactive branch is the open rule and no second validator is written.
The input reaching the refusal is an `OpenSpec` with a valid effort and model and a mode such as `Bypass` or `auto`; `TestOpenLaunch`'s new refusal rows reach it.

`buildOpenCmd` takes the resolved `skipPermissions` and, when true, appends `--dangerously-skip-permissions` to the line, after `--model` and `--effort`, on both the fresh (`--session-id`) and the resume (`--resume`) form (D4).
`OpenLaunch` passes the same resolved value to both calls, so `ResumeCmd`, the resume line or-else the fresh line, carries the flag on both branches.
The line stays one pipeline with no environment assignment, so `sh.OrElse` still joins it.
`OpenLaunch` is its only caller, on this card.

Comments: `buildOpenCmd`'s doc comment drops "no permission flag" and says it appends `--dangerously-skip-permissions` when `skipPermissions` is true, on both forms, because reed replays the resume line verbatim and a resume without it would bring a bypass session back prompting.
`Claude.OpenLaunch`'s doc comment names the permission-mode check among the checks it runs before minting a session id, and states the bound of the bypass edge: bypass is opt-in per call through `spec.PermissionMode`, the session still loads Claude Code's own settings layers (whose deny rules and hooks Claude Code applies in bypass too) and carries no shuttle `--settings` or guard, and `prompt` adds nothing, so it cannot override a default mode the settings set; the session takes no turn until text reaches its pane through the operator, `lyx shuttle send` or creel, each bounded by its own rules.
`validatePermissionMode`'s doc comment, which speaks of "a run's permission mode", says it also resolves an opened session's mode, which `OpenLaunch` passes as interactive.
The open-launch paragraph of `internal/shuttleengine/claudeengine/doc.go` adds that the line carries `--dangerously-skip-permissions` when the open's permission mode is `bypass`, on the fresh and the resume line alike, and that an unknown mode refuses the open.

Test: `TestOpenLaunch` gains rows, run through its existing loop:
- "bypass skips permissions on every line": `OpenSpec{Role: "explore", PermissionMode: "bypass"}`; the check asserts `launch.Cmd` holds `--dangerously-skip-permissions`, and that `launch.ResumeCmd` equals `sh.OrElse(resumeLine, launch.Cmd)` with `resumeLine` built from `launch.Cmd` as the existing resume row builds it, and holds the flag exactly twice (`strings.Count`) (AC1, AC2).
- "prompt adds no permission flag": `OpenSpec{Role: "explore", PermissionMode: "prompt"}`; the check asserts neither `launch.Cmd` nor `launch.ResumeCmd` holds `--dangerously-skip-permissions`, and `ResumeCmd` keeps the `sh.OrElse(resumeLine, launch.Cmd)` shape (AC3).
- "an unknown mode is refused" with `PermissionMode: "auto"` and `wantErr: "invalid permission mode"`, run through the loop's existing assertion that a refused open carries no `SessionID` (AC4).
  The wrong-case `Bypass` refusal stays pinned by `TestValidatePermissionMode`'s row, the same branch.
  Minting a session id has no side effect, so the check's place before `newSessionID` is not observable and no row claims to pin it; the refusal's place before any strand or marker directory is pinned by Card 1's `TestRunner_Open` refusal loop.
The existing "fresh line" row, which bans `--dangerously-skip-permissions` from `Cmd`, extends its ban to `ResumeCmd`, pinning the empty mode on both lines (AC3).

**ImpactSummary:** `buildOpenCmd` gains one parameter and has one caller, `Claude.OpenLaunch`, on this card; the run path's `validatePermissionMode` behaviour and `TestValidatePermissionMode` are unchanged.

**Commit:** 2: Claude's open launch realizes the permission mode on its fresh and resume lines
