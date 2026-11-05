```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, listing for each change what it forces outside the lines it touches.

**Per change, the set found and whether it is covered**

1. **`Shell.AppendLine` added to the interface.**
   - Implementers: `posixShell` and `pwshShell` only. My earlier `ExportEnv(` definition grep found no other implementer, fake or stub, and both dialects implement the method.
   - Callers: `codexengine/command.go:66` only.
   - Tests: `shell_test.go` `TestShell_AppendLine` covers both dialects. The `shell.go` and `shell_test.go` file headers were updated.
   - Covered.

2. **`Runner.SessionIdle` engine pick.**
   - Docs: the `SessionIdle` doc comment and the `shuttleengine/doc.go` pane-fallback paragraph were both updated.
   - Other comments: the remaining "set's default for a record naming none" comments are in `activity.go:74`, `sessionread.go:34`, `agentwait.go:27`, `enginesrun.go:39` and `rundir.go:157`. They describe cross-run readers that skip recordless strands, so they stay true.
   - Same-class paths: `Send` and `interruptEngine` already use the provider route, so `SessionIdle` now matches them.
   - The research doc's B3 text is a dated measurement report, so it is exempt.
   - Test: `TestRunner_SessionIdle_RecordlessStrandReadsThroughProvider`.
   - Covered.

3. **Exit record on the Codex launch line.**
   - Docs and comments: `codexengine/doc.go`, `command.go`, `codexengine.go`, `rollout.go` and `events.go` all describe the record. No remaining comment or doc still says "dead pane" alone.
   - Tests asserting the launch line: `command_test.go`, `prepare_test.go`, `skillload_test.go`, and `events_test.go` for the parse. My grep of the repo for `; exit` and `workspace-write` found no other test or doc asserting the old line.
   - Consumers of the events file:
     - `ParseEvents` ignores the record, since it keeps only Stop lines.
     - `stampEventTimes` ignores it, since `Event` is false.
     - `sessionGone` (`wait.go:1553`) and the fold's `settle` (`sessionstate.go:286`) read it as a process-ending session end. That matches the summary.
   - A hidden consumer of the unstamped line shape would be a gap, but my greps found none.
   - Covered.

4. **`sandbox_workspace_write.network_access=true` on both launch lines.**
   - Docs: `codexengine/doc.go` states what is opened and why, as the Goal requires, and `Prepare`, `OpenLaunch` and `codexCommand` comments agree.
   - Tests: `command_test.go` pins the flag in both the run line and the opened command.
   - Other references: the grep found none in docs, stencils or heddle text outside `docs/research`, whose `network_access: false` entries are dated reports and exempt.
   - Covered.

5. **PATTERN and `pattern/` entries.**
   - `PATTERN-shuttle-provider-seam`: the new `LymExit` and `codex-exit` specifics live under `codexengine`, and `session.go` reads only `strand.Provider`, so the seam holds.
   - `PATTERN-shell-guard-table`: `internal/shell` still imports only stdlib in the diff.
   - `PATTERN-test-speed`: the new tests use `tiergate.Require(t, 0)` and `t.Parallel()`.
   - The diff declares no PATTERN changes and none were needed. No `Pattern-Departure:` trailers are present or needed. The two commit messages match the diff.
   - Covered.

**Findings:** none.

**Tool calls (1 of 6)**
1. One Bash call with three greps: other tests and docs asserting the launch line's `; exit` or `workspace-write` form, other docs on the recordless `SessionIdle` engine pick, and other "dead pane" wording for Codex. My context lacked a repo-wide search for these.
