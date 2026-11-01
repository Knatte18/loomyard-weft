```yaml
lens: fix-affected-set
outcome: reported
```

Task: affected-set lens. I reran each set-premise finding's search to check that the fix covers every member.

Unchecked findings: none. I reran F1 to F4, one search each.

**[LOW] Card 13, the new failed-notice sentence after the no-verdict paragraph (13-check-fix-loop.md:50), and the TestRunCheck failed-tier row.**
- **Summary:** The same `post` call also sends the before-any-tier notice, and nothing drops that notice's `fix it, then … in the prime` way forward. That notice is a member of F2's set that the fix missed.
- **Evidence:** In `internal/adoptcli/check.go:222-226`, a run with `record.FailedTier == nil` sets `failure = record.Error` and goes on to the same `post("adopt check: failed, %s; log %s; way forward: fix it, then run %s in the prime", …)`.
- **What the plan does:** Card 13 line 49 only swaps the re-run for `lym adopt <subject>` on this end. Line 50 restricts the dropped way forward to "a failed tier".
- **Result:** A gate that exits before any tier still posts `fix it, then run lym adopt <subject> in the prime`. For a beam subject, that sends the operator to fix the wrong worktree, the same defect F2 names.
- **Test gap:** The TestRunCheck "gate exiting before any tier" row asserts only that the notice names `lym adopt <subject>`, not that `in the prime` is absent.

The other sets are covered:
- **F1:** The only plan caller of `battencli.Cancel` is card 6 (`Deps.Cancel`, `HubDeps`). It now reaches the end-only error through `cancelJudgingEndOnly`, which TestCancelRun drives.
- **F3:** Three texts say adopt starts a first pass:
  - `docs/overview.md:673` is now on card 12.
  - `internal/battenshed/doc.go:52` is on card 12.
  - `internal/beamcli/doc.go:31` is on card 11.
- **F4:** No other plan card selects reconcile outcomes by `ahead`. `reconcileWarnings` is the only site.

Tool calls:
1. Grepped `internal/adoptcli/check.go` for `in the prime` and `way forward` to find every runCheck notice. My context held only line 226.
2. Grepped `docs` and `internal` for texts saying adopt starts a first pass. My context held only overview line 673.
3. Grepped the plan cards for `ahead`/`Moved` and for `Cancel`/`EndOnly` callers. My context lacked cards 6 and 7 and the other sites.
