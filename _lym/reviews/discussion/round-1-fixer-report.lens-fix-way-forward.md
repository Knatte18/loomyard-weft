```yaml
lens: fix-way-forward
outcome: reported
```

fix-way-forward: the ways forward this fix introduced.

**Ways forward checked**
- D1's new scan, which fails on a production launch-spec literal in a package with no producer-table entry. The way forward is to add an entry.
- D4's widened scan, which fails on a hand-written mark outside `internal/shuttlecli`. The way forward is to set a prompt engine.
- D5's new refusal of a `developer_instructions` argument over 128 KiB.
- D10's step of running `Prepare` into a temporary run directory with the parity test's hook runner.
- D12's suffix-match deny and its new nudge criterion.
- D8's hook input and timeout. This is not a way forward, so I made no finding on it.

**Findings**

1. MEDIUM. Changed text: D1 "A producer outside `seatengine` takes part through its entry in the test's producer table, and a scan fails on a production launch-spec literal in a package the table has no entry for", and the first acceptance criterion's matching clause.
   Summary: the scan fails on packages whose only way forward, adding a table entry, cannot be carried out.
   Evidence: production spec literals exist in `internal/orchcli/start.go` and `internal/shuttlecli/run.go`.
   The orch is out of Scope and pinned to Claude by D4. D2 skips the orch places, so a Codex orch spec cannot pass a parity comparison.
   `lym shuttle run`'s spec is hand-written under D4: it has no seat and no prompt rendered per engine, so it has nothing to build a table entry from.
   The scan names no exemption for either package, so it fails at HEAD with no reachable fix.
   `internal/testkit/envkit/envkit.go` also builds a spec in a non-`_test` file, and the scan does not say whether that counts as production.

2. LOW. Changed text: D5 "`Prepare` refuses a launch whose `developer_instructions` argument passes the cap, naming its size and its largest item", and its new criterion.
   Summary: the refusal names no way forward, unlike its sibling D4's refusal, which names "the way forward".
   Evidence: the largest item is usually the user's global file, a CLAUDE.md above the root, or a plugin's session-start output. The refusal does not say whether to trim that file or turn the plugin off in the session scope.
   Until someone guesses, every Codex launch in that worktree stays refused.

3. LOW. Changed text: D10 "with the parity test's hook runner, which lists each plugin hook unrun".
   Summary: as worded, the step points a production verb at test code it cannot import.
   Evidence: D1's hook runner is a fake built for the `tier0` test. A cobra verb (`PATTERN-cli-cobra`) cannot import a `_test.go` helper.
   The step is reachable only if that runner lives in a non-test package, and neither D1 nor D10 says so.

Tool calls:
- Grep for `shuttleengine.Spec{` literals in non-test Go files, to learn which packages D1's new scan reaches. My context had only the `editdirective` callers, not the spec-literal sites.
- Grep of `codexengine/rollout.go` for turn and response grouping, to check that D12's per-response `apply_patch` rule can be reached. My context did not hold the rollout reader. The grep was inconclusive, so I made no finding from it.
