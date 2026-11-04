```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran the set-premise searches for F1, F3 and F6, and the fix covers every member in each. The other findings (F2, F4, F5, F7, F8) I left unchecked.

No findings.

- **F3 (callers of `commandReport` and `boundOneLine`):** `commandReport` has four callers. Two are in `capture.go` (`RunCapture` and the accepted capture) and two are in `compare.go` (timeout and non-zero exit). All go through the changed function, so all now quote the end of the tail. The only other `boundOneLine` caller is `spawn.go:186`, which keeps cutting from the end. That matches the review's instruction that `readChildReport` is unchanged.
- **F6 (timeout sites):** All three `timed out after` sites in `driftengine` now have a clause assertion:
  - the main capture (new row at `capture_test.go:205`)
  - the accepted capture (`capture_test.go:494`)
  - the compare (`compare_test.go`, new suffix assertion)

  The older main-capture subtest at `capture_test.go:332` still checks the prefix only, and the new row covers that gap. The other `timed out after` hits are in unrelated packages.
- **F1 (callers that need `lym drift capture` to succeed):** Two callers other than the rewritten subtest require `env.OK` for `capture`.
  - The third subtest at `capture_integration_test.go:221` runs after the marker file exists, so the command can't fail fast and the race doesn't reach it.
  - `anchor_integration_test.go:89` runs a command that waits for the `passed` record afterwards, so it isn't a fail-fast capture either.

Reads: one `Read` of `internal/driftcli/anchor_integration_test.go:55-104`, to check whether that capture could fail at once and race the new verb branch. The other call was a single `Bash` grep covering the `commandReport`/`boundOneLine` callers, the `timed out after` sites, and the `capture` call sites in the driftcli tests. My context held none of them.
