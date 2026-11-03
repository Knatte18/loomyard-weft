```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Findings I checked:
- **F1:** `resolveWithParentRule` has exactly three occurrences: the definition at `pullrequestmode.go:108` and its two callers at `prreview.go:93` and `finalize_pr.go:141`. Both callers pass the record-found flag, so the fix covers every caller.
- **F2:** `ReviewEnded` is read in the non-test code at `prsession.go:213`, `prreview.go:220`, `prreview.go:677-679` (closed case), `prreview.go:689` (the new open case), `prreview.go:710` and `finalize_pr.go:171`. `prreview.go:220` is the Call-entry re-entry that leads into `wait`, and `prsession.go:213` skips the session for an ended record. Neither is a judge-style verdict, so the open case at `prreview.go:689` was the only site missing the verdict.
- **F3, F4:** these two findings are test rows, not sets of sites. I did not rerun a search for them.

Unchecked: nothing.

Tool calls:
1. `grep -rn "ReviewEnded\|\.Continue\b"` over `internal` non-test Go files, to list every verdict site that reads the ending. My context held only the files the fixer read, not `prsession.go`, `loomcli` or `record.go`.

The F1 caller search came from the fixer's own `grep` already in context, so I made no call for it.
