```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the F1 fix to `_lym/landing/summary.md`, the three edits that bring it in line with the rule-read behavior.

no findings

I built three cases against the edited text. None broke.

- **Halt condition.** The summary says a Halt happens for "any other failure except an unusable token" when the run holds a record. `resolveWithParentRule` (`pullrequestmode.go:119-126`) does exactly that: a non-token, non-transient failure with `recordFound` returns `Halt`. The cancelled-context and transient errors are returned first, through `transientFailure(ctx, …)`, so the new sentence order matches the code.
- **PR-Review test claims.** Every row the Tests bullet names exists in `prreview_test.go`:
  - an unreadable rule with no record (line 168);
  - an unusable token (line 183);
  - an unreadable rule with a record (line 174);
  - a transient read failure (line 189);
  - the cancelled-context subtest (line 466);
  - an empty task branch (line 195);
  - a local origin (line 199).
  
  "An unusable origin" fits the local-origin row.
- **Finalize test claims.** The Finalize rows for the transient failure (line 108), the unreadable rule with a record (line 113) and the unusable token (line 119) exist in `finalize_pr_test.go`. "Closing nothing and keeping the record" matches the row name at line 113.

- Ran one `grep` over `prreview_test.go` and `finalize_pr_test.go` for the row names, because my context held only the review's description of them.
