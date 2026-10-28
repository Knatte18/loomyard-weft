```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens over the amendment, which is the F1–F5 fixes, the late test-ordering commit and the parent merge. For each changed behaviour I name the test that pins it, or none.

**Per behaviour: pinning test**

- **F1, lazy `programName` in `lym loom decision add`:** none. `internal/loomcli/decision_test.go:140` is the only test reference to `programName`, and it always returns `("hub:LOOM", nil)`. Two behaviours are unpinned:
  - `noticeFits` returns true when `programName()` errors (`internal/loomcli/decision.go:307-311`).
  - `preRun` (`decision.go:~780`) no longer aborts on a `hubgeom.ReedGeometry` or `agentname.FormatProgram` failure.

  The grep found no test that drives `preRun`, and none that fails the geometry for `decision add`. Reverting to the eager lookup in `preRun`, or changing the `return true` to `false`, would break no test.
- **F2, probe change in `internal/creelengine/deliver.go`:** net change is none, because commit 4d56042d reverts it. Nothing to pin.
- **F3, `tools/sandbox/SANDBOX-CREEL-SUITE.md` S9 line:** doc only; no test applies.
- **F4, no-daemon `hook-wait` exits 0 printing nothing:** pinned by the `hook-wait with no daemon to reach exits 0 printing nothing` subtest (`internal/creelcli/cli_test.go:~802`). It asserts exit code 0, empty stdout and empty stderr. The `AutoAdvance` goroutine and `done` channel from 2d9870d0 drive the client clock past the reconnect backoff so the verb can end. This is a test-only change, so nothing else is unpinned.
- **F5, an open session carries no `--append-system-prompt-file`:** pinned by the new assertion in `TestOpenLaunch` (`internal/shuttleengine/claudeengine/command_test.go:~467`), which checks both `Cmd` and `ResumeCmd`. Both lines are covered, and this is D9's open-session exclusion.
- **`traceSinkMu` ordering in `internal/fabriccli`:** test-only. It makes the existing bypass-push assertion deterministic and adds no behaviour.
- **Merge, reed `buildServerExports` in the pane launch line:** pinned by `internal/reedengine/panebin_test.go:112-155` and `internal/reedengine/spawn_test.go:467-474`. The cases cover the prelude, build-server exports, name exports and parent export in order, and the case with the prelude dropped. These tests are the parent's own.
- **Merge, `proc.NoteSpawnAgain` and `retryExecAgain`:** the parent's tests moved with the code (`internal/reedengine/spawn_test.go` and `internal/gitexec/startretry_test.go` appear in the merge stat), and nothing in the task's creel code touches them. I did not re-read them.
- **Merge, D9's `--append-system-prompt-file` together with reed's pane launch line:** no test composes the two. Composition is by construction, since the shuttle launch command is passed to `composePaneLaunchLine` as `launchCmd`, and the `TestOpenLaunch` assertion and `panebin_test.go` each pin their own half. I do not count this as a finding.
- **Merge, `latestDarnOutcome` change in `internal/loomcli/darnwiring.go`:** the parent's `TestLatestDarnOutcome` rows pin it (`internal/loomcli/wiring_darn_test.go:~409`). It is outside the task's behaviour.

**Findings**

- **LOW, class design.** Location: `internal/loomcli/decision.go:304-311` and `:780-790`, with the test gap at `internal/loomcli/decision_test.go:140`.
  - Summary: the lazily resolved LOOM program name (F1) has no failing-resolver test. The fix's behaviour (a `decision add` with no multi-entry notice never aborts on geometry, and an unresolvable program name lets `noticeFits` pass) is unpinned.
  - Suggested coverage: one row in the `decision_test.go` decision-add table. It sets `programName` to return an error, then asserts that adding a single entry records it and that `noticeFits` does not refuse.
  - Evidence: the grep over `internal/loomcli/*_test.go` showed that line 140 is the only `programName` reference, with a nil error.

**Tool calls**

1. `grep` of `internal/loomcli/*_test.go` for `preRun` and `decision add`, and of `internal/reedengine/*_test.go` for `buildServerExports`. My context lacked any test coverage of the F1 path and the build-server exports. The `programName` and `ReedGeometry` grep in the inherited context had already shown the F1 gap.
