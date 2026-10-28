```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness, over the amendment (fixes F1–F5, the late `fabriccli` commit, the parent merge). I found no defects.

**Per function or branch, with the case I ran:**
- `noticeFits` and the lazy `programName` (`internal/loomcli/decision.go:303`): with a geometry error and an empty `strandName`, it returns true and the entry is recorded. The notice posted as the program then fails on the same geometry and is reported per strand. This is the intended F1 trade-off. A successful resolve calls `LineFits`. **Correct.**
- `preRun` (`decision.go:780`): `decision add` with no notice no longer aborts on a `ReedGeometry` or `FormatProgram` error, and `programName` is called only from `noticeFits`. **Correct.**
- `decisionVerb` pre-record check (`decision.go:667`): an oversize single-entry notice is still refused before anything is recorded. It still calls `noticeFits` with the same arguments. **Correct.**
- `probe` (`internal/creelengine/deliver.go:239`): F2 and its revert net to zero against the settled code. An unmarked strand returns `(false, true)` again. **Correct.**
- `hook-wait` with no daemon (`internal/creelcli/cli_test.go:802`): the client clock advances and the reconnects run out, so it exits 0 with empty stdout and stderr. `done` closes after `awaitRun` returns, so the advance goroutine stops. `t.Setenv` makes the test non-parallel, which it states. **Correct.**
- F5 assertion (`internal/shuttleengine/claudeengine/command_test.go:467`): a negative check on the open launch's `Cmd` and `ResumeCmd`. **Correct.**
- `traceSinkMu` (`internal/fabriccli`):
  - Each anchor subtest calls `newAnchorCLIHub` once and holds the read lock until its own cleanup. No recursive read lock exists, so a waiting writer cannot deadlock it.
  - In the bypass step the `Lock` cleanup is registered before the `SetDurableSinkDir("")` cleanup. Cleanups run last-in first-out, so the sink resets before the lock releases. **Correct.**
- Parent merge (`panebin.go`, `overlay.go`, `internal/loomcli/darnwiring.go`):
  - `buildServerExports` is chained before the name exports and the launch command, so the shuttle command carrying `--append-system-prompt-file` is untouched.
  - `retryExecAgain` still returns the first non-EAGAIN result.
  - `latestDarnOutcome` touches only darn outcome reading, apart from the `decision.go` hunk. **Correct.**

No findings.

**Tool calls:**
1. `grep` of `newAnchorCLIHub`, `traceSinkMu` and `t.Parallel` in the two `fabriccli` test files — my context lacked how often each test takes the lock.
