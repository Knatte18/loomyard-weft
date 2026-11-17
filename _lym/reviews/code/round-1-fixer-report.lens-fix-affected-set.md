```yaml
lens: fix-affected-set
outcome: reported
```

Findings: no findings.

The F1 premise is a set of statements saying a failed hand-over still reaches batten. I searched for that wording in `internal` and `_lym/landing`. The only matches are the corrected `handOver` doc line at `internal/sheddriver/drive.go:353` and an unrelated line at `internal/loomcli/doc.go:611`. That line is about a verify halt, not a failed hand-over. The summary's "no longer reaches that path" sentence was the other claim, and the fixer corrected it.

Unchecked: the other `_lym` and `.md` documents outside `internal` and `_lym/landing`, such as specs and board notes. The search did not cover them.

Tool calls:
1. grep for the batten pick-up wording in `internal` and `_lym/landing`. My context had no search results for that wording.
