```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran each function and branch the diff adds or changes on a concrete input.

**Cases run**
- `hookPythonAllowedCmd` RunE, normal path in a hub worktree with `deny_python: false`: returns allowed=true, Abort is not called, `RunRootCtx` returns `es.code` 0. Correct.
- Same RunE with `deny_python: true`: returns allowed=false and err=nil, calls `Abort(1)`, exit 1, and nothing is printed. Correct.
- Same RunE with an error path (cwd outside any git repo, so `gitWorktreeRoot` returns `ErrNotAGitRepo`): prints `shuttle: <err>` to stderr and exits 1, so it fails closed. Correct.
- Same RunE when `ShouldAbort` is true: it cannot fire, because the group's PersistentPreRunE returns nil for this verb. Correct.
- `pythonAllowed` when `LoadConfig` fails on another key's validation (for example an empty `engine`): returns the error and the guard denies, so a bad config elsewhere in `shuttle.yaml` blocks Python. That is fail-closed by design and matches the summary. Correct.
- `pythonAllowed` in a standalone session (pane cwd = target, config at `StateDir`): the verb resolves the target repo's own layers or the template, not `StateDir`, and gives the wrong answer. **Finding 1.**
- `pythonAllowed` in a hub session whose Bolt checkout is not statable: `ConfigDirs` falls back to the anchor stack, the same as the launch path. Correct.
- Claude `pythonAllowedCheck`, rendered single-pattern command `grep -Eq P && ! ( cd … && lym … ) >/dev/null 2>&1 && echo DENY; true`:
  - No match: grep fails, the line short-circuits to `true`, exit 0 with no output. Correct.
  - Match with lym exiting 0: the `!` makes the step false, so nothing is printed and the hook exits 0. Correct.
  - Match with lym exiting 1, lym missing (127), or a `cd` failure: prints the deny. Correct.
- Claude multi-pattern branch: the same `deny` string is spliced after the greps. Correct.
- Codex `guardCommand` with the live check, `greps && ! check && echo DENY || echo {}`:
  - Allowed: the chain is false, so `{}` is printed. Correct.
  - Denied: the deny is printed and the `||` is skipped. Correct.
  - No match: `{}`. Correct.
- `buildDenyNotice` with `announced == nil` (the Claude-tool denies): falls back to `installed`. With a shell row, it calls `shellGuardAnnounced`. Inputs `DenyPython=false`: the python sentence is dropped while the hook stays installed. Correct.
- Codex `denyNotice` with `in.denyPython=false`: the python notice is dropped. With a row whose switch is missing from `switchAnnounced`, a nil func call would panic, but `installedGuards` panics first. Correct.
- `installedGuards` with a switch present in `switchInstalled` but missing from `switchAnnounced`: it panics, but the message says "would stay uninstalled" when the cause is the missing announcement. **Finding 2.**
- Claude `init()` with the announced map missing a switch: panics with an accurate message. Correct.
- `TestHookPythonAllowedVerb` output check `wantOutput=="" && Len!=0 || !Contains(out, wantOutput)` on the OutsideAWorktree row: `Contains(out, "shuttle: ")` holds. On the allowed and denied rows it requires empty output. Correct.

**Findings**

1. **MEDIUM, design.** `internal/shuttlecli/hookpython.go:57-65` (`pythonAllowed`).
   In a standalone run, the hook reads a different config than the launch used.
   A standalone launch loads shuttle config from `configengine.Dirs{Anchor: res.StateDir}` (`internal/burlercli/wiring.go:184`, `internal/webstercli/wiring.go:292`) but sets the pane cwd to `target` (`internal/standalonegeom/reedgeom.go:45`).
   The verb resolves `ResolveWithin(target)` and loads `fabricengine.ConfigDirs` of the target repo, which falls back to the template's `deny_python: true`.
   A standalone run configured with `deny_python: false` now gets an installed python hook that denies every Python call.
   Before this change it had no hook.
   Reproduced: run from a plain git repo outside a hub, the built verb exits 1 (`.scratch/plainrepo`).

2. **NIT, consistency.** `internal/shuttleengine/codexengine/guards.go:675-677`.
   The panic for a switch that `switchAnnounced` lacks still says "so the row would stay uninstalled", which misreports the cause.

Tool calls:
- `cmd/lym/main.go` and `clihelp.RunRoot`/`RunRootCtx`: to confirm the real binary carries the exit state, so `Abort(1)` yields exit 1. The context held only `Abort` and `ExecuteIn`.
