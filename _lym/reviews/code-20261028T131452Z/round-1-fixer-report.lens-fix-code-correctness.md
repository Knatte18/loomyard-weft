```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

I traced each changed branch against the code and found nothing wrong:

- **F1 (`decision.go`):**
  - Both callers of `noticeFits` (`decision.go:344` and `:667`) go through the new `programName()` call. `decision_test.go:138` is the only other `decisionDeps` literal, and the fixer set `programName` there.
  - When geometry fails, `noticeFits` returns true and skips the program-name length check. The record is written and each post fails on its own, reported per strand. That matches the fix's intent.
  - `preRun` still uses `output`, `clihelp`, `hubgeom` and `agentname`, so no import is left unused. The gate run was green.
- **F2 (`deliver.go`):** `probe` now returns `(false, len(snap.injects) > 0)` for a strand with neither mark. A held inject keeps its worker until the TTL expires. With no injects, `retire()` removes the worker, and a later `wake` starts a new one when mail arrives. This is the only path where the worker used to poll without end.
- **F4 (`cli_test.go`):** the new subtest runs against a rig with no daemon and a spawn that starts nothing. It would fail if `hookWait` returned 2 on the unreachable-daemon branch. It cannot hang past the 10 s `awaitRun` bound. It sets `t.Setenv`, so it is correctly non-parallel.
- **F5 (`command_test.go`):** the assertion sits in the shared `TestOpenLaunch` loop. It would fail if `OpenLaunch` started emitting `--append-system-prompt-file` on either `Cmd` or `ResumeCmd`.
- **F3:** a doc-only change, with nothing to run.

I did not run the gate and relied on the parent's green `tier0` output.

One observation, not a finding: F2 adds no test. Per the fix rule, a design finding corrects code and does not need one.

Tool calls:
1. `git diff b44df920~1 b44df920 -- internal/loomcli/decision.go`, then grep for other `programName` users. My context had the pre-edit file only. I also needed the exact committed diff, and a grep for stray consumers.
2. `grep` for `decisionDeps{` and `noticeFits(`, plus `hasPending`'s body. The first grep showed whether any other construction would hit a nil `programName`. The `hasPending` read was to confirm the worker-retire semantics. My context did not show `hasPending`.
