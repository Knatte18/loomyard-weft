```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- F1: a repo-wide grep for "dependency layers and Independent" or "dependency layers, Independent" finds only `render.go:4` and `render.go:238`. Both now name Draft, and no unfixed variant remains.
- F2: a grep for the standingCell standings list ("Ready, Note") finds only `render.go:442`, which now includes Project and Draft.
- F3: a grep for "holds the open tasks", "open task with no open dependency" and "is readyLayer" finds only `layer.go:29`, which is fixed. The `doc.go:176` phrase "holds the open tasks" no longer matches, so it was rewritten. No other copy of either wording exists outside `_lym`.

Tool calls (2 of 6):
1. Grep for the Tasks-subsection list wording (F1), because my context showed only the two lines the review named.
2. Grep for the Ready-sentence wording and the standingCell standings list (F2, F3), for the same reason.
