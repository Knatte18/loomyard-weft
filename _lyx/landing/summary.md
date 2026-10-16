# lyx shuttle open takes --permission-mode

`lyx shuttle open` gains a `--permission-mode` flag, so an operator session opens in bypass under its lyx strand name (`<shortname>[:<slug>]:<role>`) instead of being removed and relaunched through `lyx reed add` under a generic name.

Values are `bypass` and `prompt`.
Empty, the default, keeps today's behaviour: the launch line carries no permission flag.
There is no config default for the mode.

## What changes

- `shuttleengine.OpenSpec` gains `PermissionMode`, caller-owned engine vocabulary like `Spec.PermissionMode`.
  The CLI and `Runner.Open` pass it through uninspected, and `Runner.Open` logs it on the "opened session" line so a bypass open is traceable.
- `claudeengine`'s `OpenLaunch` validates the mode through `validatePermissionMode(mode, true)` before it mints a session id.
  `bypass` adds `--dangerously-skip-permissions`; `prompt` and empty add nothing; any other value is refused with the existing error naming the legal values.
- `buildOpenCmd` puts the flag on both the fresh and the resume line, so `ResumeCmd` keeps bypass on the resume branch and on the fresh fallback.
  Without it a resumed bypass session would come back prompting.
- The `open` help text, the `shuttleengine` and `claudeengine` package docs, and the shuttle row of `docs/overview.md` describe the flag.

## Design decisions to review

- The `bypass` to flag mapping lives only in `internal/shuttleengine/claudeengine`; `shuttleengine` and `shuttlecli` carry the string blind (`PATTERN-shuttle-provider-seam`).
- `prompt` adds no flag, so it cannot override a default mode the Claude Code settings set.
- Refusals `Runner.Open` makes before `OpenLaunch` (unknown engine, covering pause, engine without `SessionOpener`) still report ahead of a mode refusal.
- Bypass is opt-in per call.
  The session takes the worktree's own Claude Code settings, whose deny rules and hooks still apply; lyx adds no `--settings` or shuttle guard, as today.
  An opened bypass session runs every tool without a dialog once text reaches its pane, through the operator's window, `lyx shuttle send`, `lyx creel inject` or a `lyx creel post` reminder.
  Each channel keeps its own rules, and a covering pause holds all of them.

## To check by hand

- The `ly:cli` index lists commands, not flags, so the orch finds the flag only through `lyx shuttle open --help`.
  Until it reads that, it may still fall back to `lyx reed add` for a bypass session.
- Test coverage of the `open` refusal envelope: confirm `internal/shuttlecli/cli_test.go` still exercises an unknown-mode refusal end to end, and that the refusal happens before a strand is added or the marker directory is created.
- The integration tier and a real open with `--permission-mode bypass` were not run here; start one and check the launch line carries `--dangerously-skip-permissions`.
