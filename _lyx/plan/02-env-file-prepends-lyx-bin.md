# Card 2 — The Claude engine's env file re-prepends LYX_BIN's directory

**Create:**
- `internal/shuttleengine/claudeengine#envFileContentPosix`
- `internal/shuttleengine/claudeengine#envFileContentWindows`
- `internal/shuttleengine/claudeengine#envFileContentFor`

**Delete:**
- `internal/shuttleengine/claudeengine#envFileContent`

**Edit:**
- `internal/shuttleengine/claudeengine#Claude.Prepare`
- `internal/shuttleengine/claudeengine#TestPrepare_WritesArtifactsAndReturnsConsistentLaunch`
- `internal/shuttleengine/claudeengine#TestEnvFile_DefaultStdinIsDevNull`
- `internal/shuttleengine/claudeengine/doc.go`
- `internal/reedengine/doc.go`

**Uses:**
- `internal/agentname#LyxBinEnv`

**Intent:**
D5 and D7.
The env file's content becomes a per-OS constant, chosen by `envFileContentFor`, which returns `envFileContentWindows` for `goos == "windows"` and `envFileContentPosix` otherwise.
Both constants are Go constant expressions built from `agentname.LyxBinEnv`, so the Claude engine reads the same exported key reed's prelude exports (D6), and both start with today's `exec </dev/null` line unchanged.
The declaration heads above carry an empty placeholder literal only to name the symbol; the real values are:

- `envFileContentPosix`: `exec </dev/null`, then one statement that, when `LYX_BIN` is set and non-empty, prepends `LYX_BIN`'s parent directory to `PATH`, every expansion double-quoted so a path with spaces holds (for example `if [ -n "${LYX_BIN:-}" ]; then PATH="${LYX_BIN%/*}:$PATH"; fi`).
- `envFileContentWindows`: `exec </dev/null`, then one statement that, when `LYX_BIN` is set and non-empty and `command -v cygpath` succeeds, converts `LYX_BIN` with `cygpath -u` into the scratch variable `__lyx_bin_dir`, prepends that path's parent directory to `PATH`, quoted the same way, and unsets `__lyx_bin_dir`; with `cygpath` absent or `LYX_BIN` unset or empty it changes nothing.

Each line ends in a newline, and neither constant runs anything else.
The statement reads `LYX_BIN` when Claude Code runs the file before each Bash command, so a resumed session follows the binary its pane's latest prelude exported.

`Claude.Prepare` writes `envFileContentFor(runtime.GOOS)` instead of the deleted `envFileContent`.
`TestPrepare_WritesArtifactsAndReturnsConsistentLaunch` compares `bash-env.sh` against `envFileContentFor(runtime.GOOS)`.

Bound: the statement prepends only `LYX_BIN`'s parent directory, the directory reed's pane prelude already puts first in the pane shell, so the Bash tool shadows exactly the executables the pane shell shadows.
It adds no directory when `LYX_BIN` is unset or empty or, on Windows, when `cygpath` is absent; it grants or denies nothing, so the agents' permission rules are untouched.
`PATTERN-shuttle-provider-seam` keeps the change under `internal/shuttleengine/claudeengine`, and `PATTERN-shell-mechanics-seam` does not bind it: the file is a script Claude Code sources, not a pane-shell command string.

`TestEnvFile_DefaultStdinIsDevNull` (integration tier, `envfile_integration_test.go`) becomes the env file's one behavior test: its table gains the content each row runs, picked through `envFileContentFor("linux")` or `envFileContentFor("windows")` so the selector's two branches are reached, and an explicit environment per row; the `bash -c '<content>eval <command>'` shape and the open-pipe stdin stay.
Its three stdin rows run the POSIX content with `LYX_BIN` removed from the environment, so they also pin that the content with `LYX_BIN` unset still makes `/dev/null` the default stdin.
The `PATH` rows build a temp directory holding a decoy executable `lyx` and a second directory whose name contains a space holding the real executable `lyx`, run with `PATH` set to the decoy directory prepended to the test process's `PATH`, the order an operator rc leaves behind, and skip on a Windows host.
New rows, each asserting exact stdout:

- POSIX content, `LYX_BIN` naming the spaced path: `command -v lyx` prints `LYX_BIN`.
- POSIX content, `LYX_BIN` unset: `printf %s "$PATH"` prints the given `PATH` unchanged.
- POSIX content, `LYX_BIN` empty: `printf %s "$PATH"` prints the given `PATH` unchanged.
- Windows content, `LYX_BIN` naming the spaced path, no `cygpath` on `PATH`: `printf %s "$PATH"` prints the given `PATH` unchanged.
- Windows content, `LYX_BIN` unset, a stub `cygpath` on `PATH`: `printf %s "$PATH"` prints the given `PATH` unchanged.
- Windows content, `LYX_BIN` naming the spaced path, a stub `cygpath` (a temp executable script printing its last argument) in a directory on `PATH`: `command -v lyx; printf %s "${__lyx_bin_dir-unset}"` prints `LYX_BIN`, a newline, then `unset`.

Each row would fail if the statement were missing, unquoted, ran unconditionally, ran on an empty value, skipped the conversion, or left its scratch variable behind.
The test's doc comment and the file's header say the file pins both the default stdin and the `PATH` re-prepend.

Docs: `internal/shuttleengine/claudeengine/doc.go`'s env-file paragraph replaces "The file runs that one statement and nothing else" with the two statements' content and the bound above, and says the PATH statement reads `agentname.LyxBinEnv`.
`internal/reedengine/doc.go`'s pane-binary paragraph (D7) adds that a fresh shell a strand's command starts re-runs its rc and can reorder `PATH`, that `LYX_BIN` names the spawning binary there, and that a shuttle engine re-asserts `LYX_BIN`'s directory in its agent's tool shell.

**ImpactSummary:** One unexported constant replaced by a per-OS pair and a selector inside `internal/shuttleengine/claudeengine`; its one production reader is `Claude.Prepare`, and its exact-content test and its behavior test are updated; reed's code is untouched, only its package doc.

**Commit:** 2: Re-prepend LYX_BIN's directory in the Claude engine's Bash env file
