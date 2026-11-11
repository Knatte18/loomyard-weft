```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals F1 and F2 as the fixer carried them out.

**Findings**

1. **LOW**, `internal/reedcli/cli_test.go`, `TestRunCLI_Down_IsNotRefusedInBolt`: F2's proposal was only half carried out, and the missing half is the one the review cared about.
   - The review asked for rows proving attach and up still pass the guard in an ordinary task worktree. A guard that over-refused a task worktree would still pass tier0.
   - The fixer dropped those rows because they start a real tmux server (the tier0 and tier2 runs reported leaked servers).
   - The smoke tests do run `up` in a prime fixture (`internal/reedcli/smoke_debuglog_test.go:59`, `mustRunReed(t, prime, "up")`), but nothing exercises `attach`, and nothing covers a task worktree as opposed to the prime.
   - The fixer-report must list this as a deferral with that reason, not as fixed.
   - The fixer's own reasoning supports the deferral.
   - A cheaper alternative would be a tier0 test that calls the guard directly. It cannot be done without changing `internal/reedcli/cli.go:166`, where the check is inline in the pre-run. Extracting it would leave the fixer's write surface for a test-only finding.

2. **LOW**, `cli_test.go`: `fabricRefusalText` duplicates a literal that the existing `TestRunCLI_AttachAndUp_RefuseFabricCheckouts` still hardcodes.
   - The constant holds `"boots a session only in a worktree of the hub"`. The existing test builds the same text inline at line 125, in `"reed: lym reed " + tt.verb + " boots a session only in a worktree of the hub"`.
   - The two copies can drift apart, and a drift would silently make the new down test vacuous.
   - The file header comment (lines 1–7) also does not mention the new down test.

**F1 challenge:** no finding.
- The new `TestRunCheckMarks` row, a green tier then exit 1, reaches the `record.State == checkFailed` branch with `FailedTier` nil.
- With that branch removed, the case would fall through to `default` and render a `checkRunning` mark, so the row fails with the defect in place.
- The gate's tier0 run passed with the row present.

**F2's down row:** it fails if the guard over-refuses down, because the refusal text would appear in the output. It passes trivially when down prints nothing, which is its intended assertion.

**Tool calls beyond context:** none.
