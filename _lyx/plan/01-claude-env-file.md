# Card 1 — Stdin comes from CLAUDE_ENV_FILE, not a hook

**Edit:**
- `internal/shuttleengine/claudeengine/settings.go`
- `internal/shuttleengine/claudeengine/command.go`
- `internal/shuttleengine/claudeengine/claudeengine.go`
- `internal/shuttleengine/claudeengine/doc.go`
- `internal/shuttleengine/claudeengine/settings_test.go`
- `internal/shuttleengine/claudeengine/command_test.go`

**Delete:**
- `internal/shuttleengine/claudeengine/bashstdin_integration_test.go`
- `internal/shuttlecli/smoke_bashstdin_test.go`

**Create:**
- `internal/shuttleengine/claudeengine/envfile_integration_test.go`

**Uses:**
- `internal/shell/shell.go`

**Intent:**
Decision record D1: every lyx-spawned Claude session gets `/dev/null` as the default stdin of its Bash tool's shell through Claude Code's own `CLAUDE_ENV_FILE`, and the PreToolUse(Bash) stdin rewrite hook is removed.

`settings.go`: delete `bashStdinPrefix`, `bashStdinRewriteJSON`, `bashStdinHookCommand` and the unconditional Bash hook entry `buildSettings` appends after the webster fork guard.
The fork guard (fork runs only), the Agent and AskUserQuestion hooks and the Stop hook stay exactly as they are.
Rewrite the file header's rewrite-hook paragraph: drop it and say the default stdin is set by the env file `Prepare` writes (see `command.go`), not by a hook.

`command.go`: add a const for the variable name `CLAUDE_ENV_FILE`, a const for the file name `bash-env.sh`, and a const for its content, `exec </dev/null` followed by a newline, each with a comment.
Give `buildLaunchCmd` and `buildResumeCmd` an `envFilePath string` parameter and wrap the finished line in `sh.WithEnv(<the CLAUDE_ENV_FILE const>, envFilePath, cmd)` after the fork-subagent wrap, so the assignment leads the line, e.g. posix `CLAUDE_ENV_FILE='/run/bash-env.sh' CLAUDE_CODE_FORK_SUBAGENT='1' 'claude' …` and pwsh `$env:CLAUDE_ENV_FILE = 'C:\run\bash-env.sh'; $env:CLAUDE_CODE_FORK_SUBAGENT = '1'; & 'claude' …`.
The line stays single-line, and every quoting goes through `internal/shell` (`PATTERN-shell-mechanics-seam`).
Update both functions' doc comments: the variable rides the pane line for the same reason `CLAUDE_CODE_FORK_SUBAGENT` does, and the resume line carries it so a resumed session keeps the default stdin.

`claudeengine.go` (`Claude.Prepare`): after writing settings.json, write the env file into `runDir` beside it with the content const (0o644), resolve its absolute path with `filepath.Abs`, and pass that path to both builders.
The env-file path stays the native absolute path on every OS; Claude Code reads the file itself and runs its content in the Bash tool's shell (git-bash on Windows, where `exec </dev/null` is valid).
Update `Prepare`'s doc comment to name the third artifact.

Bound, stated in the `command.go` const comment: the file runs one `exec </dev/null` and nothing else.
It changes only the default stdin of the tool's own shell: a heredoc, pipe or redirect attached to a command still supplies that command's stdin, a child `bash` the agent starts does not re-run the file, and it grants or denies nothing, so the agents' permission rules are untouched.
A `CLAUDE_ENV_FILE` the operator's own environment exports is overridden for lyx-spawned sessions only.

`doc.go`: replace the rewrite-hook paragraph with the env-file description: the file, its content, the variable on the launch and resume lines, and the bound above.
The file and the variable name are provider specifics and stay inside `claudeengine` (`PATTERN-shuttle-provider-seam`).

Tests:
- `settings_test.go`: replace `TestBuildSettings_BashStdinRewrite` with a test over the same run-mode matrix (autonomous, interactive, fork, fork with allow-agent) asserting that the only Bash PreToolUse entry a document carries is the webster fork guard in a fork run, and that a non-fork run carries no Bash entry at all.
  Extend `TestPrepare_WritesArtifactsAndReturnsConsistentLaunch`: `bash-env.sh` exists in the run directory with exactly the content const, and both `Launch.Cmd` and `Launch.ResumeCmd` carry `shell.ForGOOS().WithEnv` of the variable over the file's absolute path as their leading assignment.
- `command_test.go`: thread an env-file path through every `TestBuildLaunchCmd`, `TestBuildResumeCmd` and `TestBuildResumeCmd_NoticePosix` row and update each exact `want` to its new leading assignment, for the posix and pwsh shells alike.
- Delete `bashstdin_integration_test.go`, which pins the deleted hook, and `internal/shuttlecli/smoke_bashstdin_test.go`, the live proof that the deleted hook kept permission rules in charge; with no hook nothing rewrites a command, so there is nothing left for it to prove.
- `envfile_integration_test.go` (`//go:build integration`, spawns `bash`): run `Prepare` into a temp run directory, read the env file it wrote, and run `bash -c` in Claude Code's prelude shape — the file's content, then `eval '<cmd>'` with the command shell-quoted — with the shell's stdin an `os.Pipe` whose write end the test holds open.
  One table-driven test with three rows: a bare `cat` ends at once with empty output (a bound like the deleted test's kill timer fails the row rather than hanging); a heredoc attached to `cat` still prints its body; `printf data | bash -c 'cat'` still prints `data`.
  It calls `t.Parallel`; the package's existing `TestMain` already covers tmux and git isolation.

**ImpactSummary:** every Claude session lyx spawns loses its PreToolUse(Bash) stdin hook and gains `CLAUDE_ENV_FILE` on its launch and resume lines plus a `bash-env.sh` in its run directory; the fork guard and every other hook are unchanged.

**Commit:** 1: claudeengine: default Bash stdin from CLAUDE_ENV_FILE, drop the rewrite hook
