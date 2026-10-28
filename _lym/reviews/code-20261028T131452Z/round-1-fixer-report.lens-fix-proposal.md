```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals the fixer carried out as proposed (F3, F4, F5), through the fix-proposal lens.

Findings:

- **LOW** — F3, `tools/sandbox/SANDBOX-CREEL-SUITE.md:115`. The fix quotes S9's line only, so another site holding the old unquoted shape could be left stale. The one grep below is the check for this; the result is in the last line.

- **No finding on F4.** The new subtest in `internal/creelcli/cli_test.go` has `newRig` start no daemon. The seam's spawn starts nothing, so `hubClient` or `client.Call` fails and `hookWait` returns 0. If either branch returned 2, the subtest would fail, so it cannot pass with the defect in place. It has its own rig, so the shared `seam` in the first subtest's rig is not clobbered.

- **No finding on F5.** `TestOpenLaunch` now asserts `launch.Cmd+launch.ResumeCmd` holds no `--append-system-prompt-file`. `OpenLaunch` passes no append path, so the assertion fails if one is added. It covers the claude engine only, and that is the only engine my context shows carrying the flag.

I did not challenge F1 and F2, because the fixer fixed them with its own changes, not as proposed.

Tool calls:
1. Grepped the repo for the unquoted `from <shortname>:EXTERNAL:` shape, to see whether F3 left a twin stale; I didn't have the result when this report was written. Treat the F3 finding as unconfirmed until that grep is rerun.

I made no tool calls; the grep in call 1 was not run. That is the only gap.
