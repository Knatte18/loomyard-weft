```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each change in the diff to the goal points of the decision record, which serves as the answer key on its own here.

**Decisions** (the record has no plan and no `Added after Discussion` entries)
- G1: put the run's slug in the verify-budget line, the way other parent notices do, so the parent knows which run it is about without reading its status. Built by C1.

**Changes**
- C1: `internal/loomcli/darnwiring.go:9,18` (diff lines). The verify budget's `Notify` seam in `treeScopeSettings` now opens the line with `loom <slug>: `, and its doc comment gains a line saying so. Traces to G1.
  - The board entry's Seen section names `verifyBudgetNotice` in `internal/loomshed/verifybudget.go` as where the line is built. Its Idea section asks only that the slug appear in the line, not where it is added.
  - Adding it at the wiring seam matches the "as other parent notices do" form: `raddlewiring.go:246` and `wiring.go:995` both use `fmt.Sprintf("loom %s: %s", ...)`.
- C2: `internal/loomcli/notifystamp_integration_test.go`. This adds the new `TestVerifyBudgetNotice_NamesTheRun` and widens the file header to cover verify-budget notices. Forced by C1, since it is the test of the behaviour C1 adds and the header that C1's new test makes stale.

**Findings**
None. Every change traces to G1 or is forced by C1, and G1 is built.

Tool calls: none. My inherited context already held the diff, the decision record, the board entry and the sibling notice sites.
