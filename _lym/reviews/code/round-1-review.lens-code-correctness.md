```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens. I traced each function and branch the diff adds or changes with concrete inputs.

**Per function or branch: the case I ran, and whether it is correct**

- **`hookPythonAllowedCmd` RunE** (`internal/shuttlecli/hookpython.go:41-53`):
  - Allowed, no error: `pythonAllowed` returns (true, nil), nothing is printed, no `Abort`, exit 0. Correct.
  - Denied: returns (false, nil), `Abort(ctx, 1)`, exit 1, stdout empty. Correct.
  - Error: returns (false, err), `output.Err` writes to stderr, then `Abort` gives exit 1. Correct, and the error is reported, not swallowed.
  - Extra argument: `cobra.NoArgs` errors and RunRoot exits non-zero. The hook denies. Correct.
  - Aborted pre-run: the shuttle group's pre-run returns early for this verb (`cli.go:81`), and the root pre-run never aborts. The `ShouldAbort` guard is just the convention. Correct.
- **`pythonAllowed`** (`hookpython.go:58-73`):
  - `CwdFrom` with no ctx value falls back to `lymcwd.Getwd`; its error propagates. Correct.
  - `shuttleConfigDirs` error propagates. Correct.
  - `LoadConfig` error (unparsable YAML at the default layer) propagates and the hook denies. Correct.
  - Config absent at every layer: the template gives `DenyPython` true, so exit 1. That matches what such a launch saw. Correct.
- **`shuttleConfigDirs`** (`hookpython.go:79-94`):
  - Hub prime worktree with a Bolt checkout: `ResolveWithin` succeeds, `ConfigDirs.Board` is set, and the hub's layers come back. These are the same dirs `orchcli.wireCLI` loads (`orchcli/cli.go:223`). Correct.
  - Directory below the anchor: `ResolveWithin` accepts it (`lymcwd.go:158`), giving the hub layers. Correct.
  - Directory outside any repository: `gitWorktreeRoot` returns `ErrNotAGitRepo`, and `RepositoryRootOf(cwd)` returns cwd.
    The state dir derived for that path does not exist, so the result is (zero Dirs, `ErrNotAGitRepo`). Correct: it fails closed.
  - Standalone target that is a git repo:
    - `ResolveWithin` succeeds, since the anchor falls back to `.`.
    - `ConfigDirs` leaves `Board` empty, because no `_bolt` sits beside the target.
    - `RepositoryRootOf(cwd)` gives the target, and `Derive` normalizes it the way the launch's `RepositoryRootOf(Normalize(told))` does (`standalone.go:200`).
    - With the state dir present, `Dirs{Anchor: stateDir}` comes back. That matches webster's and burler's standalone `LoadConfig` (`webstercli/wiring.go:292`, `burlercli/wiring.go:184`). Correct.
  - Standalone target whose state dir is absent: returns (`hubDirs` with an empty Board, nil). `LoadConfig` falls back to the template, so the run denies. Such a run never launched through standalone wiring anyway. Correct.
  - Hub worktree whose Bolt checkout cannot be statted: Board is empty. Unless a standalone state dir happens to exist for that worktree root, it returns the anchor and records layers, as the launch's `ConfigDirs` did. Correct. The other ordering applies only to a degenerate hub, so I raise no finding.
  - Hub cwd under the worktree but outside a non-`.` anchor: `ResolveWithin` errors, and the standalone state dir is normally absent. The original error comes back and the hook denies. Correct: it fails closed.
- **`cliwire.StandaloneStateDir`** (`standalone.go:55`): a relative target makes `Derive` return `errRelativeTarget`. The caller only passes absolute paths, because `CwdFrom` and `RepositoryRootOf` keep them absolute. Correct.
- **Claude `renderShellGuard` with a live check** (`settings.go:406-431`):
  - Single-pattern branch: `grep -Eq p && ! (cd "$P" && lym …) >/dev/null 2>&1 && echo deny; true`. A match plus lym exiting 0 prints nothing. A match plus lym exiting 1, lym missing, or `cd` failing prints the deny. No match never runs lym. The command always exits 0. Correct.
  - Multi-pattern branch: same shape after `in=$(cat)`. Correct.
  - Rows with no live check render exactly as before. Correct.
- **Claude `buildDenyNotice`** (`settings.go:437-449`):
  - Python row with `cfg.DenyPython` false: installed is true, and the announced predicate returns false, so there is no sentence. Correct.
  - Deny with a nil `announced` (the Claude-tool denies) falls back to `installed`, which keeps the old behaviour. Correct.
- **Claude `init` check** (`settings.go:612`): a switch missing from the announced map panics at init. Correct.
- **Codex `installedGuards`** (`guards.go:66-79`):
  - Inputs with `denyPython` false: the python row is installed. Correct.
  - Unknown switch in either map: panics. Correct.
- **Codex `guardCommand`** (`guards.go:91-100`):
  - A match plus lym exiting 0: `! check` fails, `|| echo '{}'` prints the empty output. Correct.
  - A match plus lym exiting 1 or missing: the shell's "not found" goes to `/dev/null` through the command's own redirect, and the deny prints. Correct.
  - No match: prints `{}`. Correct.
- **Codex `denyNotice`** (`guards.go:136-147`): with rows from `installedGuards`, every switch is known, so indexing the map is safe. `denyPython` false drops the python notice only. Correct.
- **Codex `Prepare`** (`codexengine.go:168-175`): `guardIn` is built once and used for both install and announce. Correct.
- **Root pre-run on the hook's process** (`cmd/lym/main.go:152-183`): the network pause gate only gates network operations. It does not block a config read, so the hook cannot hang on a paused hub. Correct.

**Findings**

None. Every branch and error path I ran gives the expected result, fails closed where it should, and reports its errors.

**Tool calls**

1. `internal/tendercli/gate.go` `InstallNetworkGate`: checked whether the root pre-run could block or slow the hook's `lym` process. My context held only the call site at `cmd/lym/main.go:120`.
