```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test row that pins it.

**Per behaviour, the test that pins it**

1. **`runHubStop` step order, with the prime's session last.** `TestRunHubStop`, row "an owned pause is lifted after the reap…" (`internal/beamcli/hubstop_test.go:438`).
2. **`runHubStop` lifts only an owned entry; a standing entry is left.** `TestRunHubStop`, rows 1 and 2 (`hubstop_test.go:438-443`).
3. **`runHubStop` goes on past an unconfirmed pause and names what was busy (`ConfirmationTimedOut`).** `TestRunHubStop`, row 3 (`hubstop_test.go:446`).
4. **`runHubStop` maps `ConfirmationLifted` to its fixed `Unconfirmed` text.** None.
5. **`runHubStop` goes past a failed step.**
   - Failed session down: pinned by row 4 (`hubstop_test.go:450`).
   - Failed pause, worktree list, `beamDown`, reap and lift: none. No row exercises any of them, so nothing pins that a lift error leaves `PauseLifted` false or that each failure is recorded and the stop goes on.
6. **`hubStopWorktrees` sorts beams from task pairs and skips the main, prunable and staging worktrees.** None. The test replaces `worktrees` with a fake (`hubstop_test.go:467`).
7. **`stopAll` refuses a slug given beside `--all` and names both ways forward.** None. The only "exclusive" row in the refusal table is for `start` (`internal/beamcli/start_integration_test.go:72`).
8. **`stopAll` in the caller's process: spawns the detached process and prints its pid; a failed spawn names a way forward.** None.
9. **`stopAll` in the detached process: a failure turns into the envelope error and the log line.** None.
10. **`spawnDetachedStop` builds its args and working directory (`beam stop --all --detached` in the prime).** None.
11. **`hibernatecli.PauseHub` passes the caller label and requester through.** None. This wiring is thin.
12. **`PauseAndConfirm` with no request file: owned entry under the caller's label, standing entry covers, timeout names what was withheld.** `TestPauseAndConfirm` (`internal/hibernateengine/pauseconfirm_test.go:603-613`).
13. **`carry` through `PauseAndConfirm`: None/Lifted/Stopped/TimedOut map to fail, cancel, withdraw and fail.** Existing carry rows:
    - pause error: `watch_test.go:236`;
    - lifted: `watch_test.go:186-215`;
    - timeout: `watch_test.go:334`;
    - cancel during the wait: `watch_test.go:489`.
14. **`beamDown` ends the orch process last and reports `OrchProcessEnded`.** `TestBeamDown_StopsOrchThenTheDobbyThenTheOrchProcess`, all rows (`internal/orchcli/beamdown_test.go:901-906`).
15. **`beamDown` wraps an `endOrch` error.** None.
16. **The `down` refactor in reedcli (verb envelope unchanged).** Covered by the existing down verb tests (`internal/reedcli/down_test.go`).
17. **`DownWorktree` wiring.** None directly. `wireLocation` is shared with the verb's pre-run, so the verb tests cover it.
18. **Prime-group `stop` button entry.** `TestLoadConfig_TemplateDefaultsResolve` (`internal/reedengine/config_test.go:117`), for the running OS's template.

**Findings**

- **MEDIUM | design | `internal/beamcli/hubstop.go:207-210`.** The new refusal of a slug beside `--all` has no test that reaches it and asserts its message. Evidence: the refusal is at `hubstop.go:209`; the refusal table in `start_integration_test.go:55-80` holds a `start b --all` row but no `stop` row. Fix: add a `stop b --all` row to that table.
- **MEDIUM | design | `internal/beamcli/hubstop.go:295-323`.** `hubStopWorktrees`, which decides which worktrees count as task pairs and get their processes and sessions downed, has no test. A wrong split would down a beam as a task pair or skip a pair, and no row would fail. Evidence: the test fakes `worktrees` at `hubstop_test.go:467-469`.
- **MEDIUM | design | `internal/beamcli/hubstop.go:350-390`.** Five of the six failure paths in `runHubStop` are uncovered: pause, worktree list, `beamDown`, reap and lift. So is the `ConfirmationLifted` mapping. Nothing pins that a lift error leaves `PauseLifted` false and is named, or that a `beamDown` error is recorded while its report is still appended. Evidence: `hubstop_test.go:437-453` holds only a session-down failure; the fakes at `463-489` never return an error from pause, `beamDown`, reap or lift.
- **LOW | design | `internal/beamcli/hubstop.go:207-256`.** `stopAll`'s two modes have no coverage: the caller prints the detached pid, and a failed spawn becomes an error naming a way forward. `spawnDetachedStop`'s argument list (`--all --detached`, working in the prime) is uncovered as well. A dropped `--detached` would make the detached process spawn itself again, and no row would fail. Evidence: `hubstop.go:220-226, 244`; nothing under `internal/beamcli/*_test.go` calls `stopAll` or `spawnDetachedStop` (grep).
- **LOW | design | `internal/orchcli/stop.go:74-77`.** No row makes `endOrch` fail, so the wrapped error and the partial report it returns are unpinned. Evidence: the `endOrch` fake at `beamdown_test.go:914-920` never returns an error.
- **NIT | design | `internal/beamcli/hubstop_test.go:503`.** The failure row asserts only the number of failures. It does not check that the failure names `t1`'s session down, or that the failed down is left out of `report.Pairs`. Evidence: `hubstop_test.go:503-505`.

**Tool calls**

1. Grepped the hibernate, beamcli and reedcli tests for carry outcome rows, `stop`/`--all` refusal rows, down tests and callers of the new helpers. My context did not list which existing tests cover these.
2. Read the carry failure rows in `watch_test.go`, the refusal table in `start_integration_test.go:55-100`, and grepped for stop and `OrchProcessEnded` tests. I needed to confirm which rows exist and that no `stop --all` refusal row does.
