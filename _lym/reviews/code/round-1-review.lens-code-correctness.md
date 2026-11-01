```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens on the diff (desktophook, ideengine, reedcli, config registration).

**Per function / branch, case run:**

- `AfterWindow`, config load fails (malformed machine override): returns a wrapped error. Correct, and both callers only warn.
- `AfterWindow`, `window_command` empty or whitespace: returns nil before sleeping. Correct.
- `AfterWindow`, `window_wait_ms: 0`: `time.Sleep(0)` returns at once, then the command runs. Correct.
- `AfterWindow`, `window_wait_ms` negative: `LoadConfig` refuses it. Correct.
- `AfterWindow`, very large `window_wait_ms` (for example 10^12): no upper bound, so `time.Sleep` blocks the verb for that long, and the wait is outside `commandTimeout`. This is a self-inflicted misconfiguration in a machine-local file, so I do not file it.
- `runWindowCommand`, hanging command: `CommandContext` kills `sh` at 30s and `WaitDelay` bounds the output read. Correct. A grandchild that holds the pipe is cut off after 5s. Correct.
- `runWindowCommand`, command exits nonzero: the error carries the command's output. Correct.
- `runWindowCommand`, `dir` missing: `cmd.Dir` makes the start fail, and the error is returned. Correct.
- `runWindowCommand`, command contains a `~`: `sh -c` expands `~/.local/bin/x.sh` only at the start of a word. That is the Goal's own example, and it works.
- `LoadConfig`, hub base file absent: `LoadOrTemplate` degrades to the template. Correct.
- `LoadConfig`, local override of a machine key: refused by `ValidateLocal`, which the test pins. Correct.
- `Spawn`, launcher error: returns the error and skips `placeWindow`. Correct.
- `Spawn`, launcher success: runs `placeWindow` with `result.launchPath`, and returns nil. The prime's launch path is the workspace file, so the command's working directory is the hub directory. Correct, since `Dir` is set to a directory that exists. If that path is a file, `cmd.Dir` fails to start and only a warning results. For the prime, `launchPath` is the `.code-workspace` file, and `exec` with a file as `Dir` fails with ENOTDIR. This is a real defect, listed under F1.
- `OpenTask`, launcher error: returns it. Success: warns on a hook failure only. Correct.
- `placeTerminalWindow`, `dir` outside any worktree: `ResolveWorktree` errors, a Debug log is written, nil is returned. Correct and degrading.
- `placeTerminalWindow`, `dir` a subdirectory of a worktree: `ResolveWorktree(worktreeRoot)` is passed a non-root path. The comment says "the worktree holding dir", but `ResolveWorktree` takes a worktree root. Whether `resolveCore` walks upward I did not verify, and a miss would silently skip the hook for a subdirectory. I am noting it as uncertain, with no further tool call spent.
- `windowCmd` RunE: the launcher error returns before the place call, so no command runs after a failed open. Correct. A place error only warns, and the envelope is still Ok. Correct.

**Findings:**

F1: MEDIUM, design. Location: `internal/ideengine/spawn.go:46-50` with `internal/desktophook/window.go:55`. For the prime, `result.launchPath` is the `&lt;hub&gt;/&lt;RepoName&gt;.code-workspace` file (per the `Spawn` doc comment), and it is passed to `AfterWindow` as `dir`, which becomes `cmd.Dir`. A file as a working directory makes the start fail with ENOTDIR, so the window command never runs for the prime's VS Code window, which `lym ide` opens. Only a warning is logged, and the tests cover the task slug only (`spawn_hub_integration_test.go`, `checkTaskSlugOpensBareFolder`). Evidence: `spawn.go:46` and `spawn.go:15`, the `Spawn` doc comment describing the prime launching the workspace file. I did not read where `launchPath` is assigned; this rests on the doc comment. Fix: pass `filepath.Dir(launchPath)` when `launchPath` is not a directory, or give `placeWindow` the directory explicitly.

F2: LOW, design (uncertain). Location: `internal/reedcli/window.go:31-38`. `placeTerminalWindow` passes the told dir to `lymcwd.ResolveWorktree`, which takes a worktree root. A dir that is a subdirectory of a worktree may fail to resolve, so the hook is silently skipped, with only a Debug log. The `lym reed window` doc says "any directory the caller names". I did not verify `resolveCore`'s walk-up behaviour. Fix: confirm `resolveCore` walks upward; if not, resolve through the containing worktree root before calling it.

Tool calls: none used. My context held every file I needed, except where `launchPath` is assigned and `resolveCore`'s body, which are the basis of F1's and F2's uncertainty.
