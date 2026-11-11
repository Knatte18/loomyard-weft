```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun each finding's set search (lym steps that name a worktree root) and check the fix covers every member.

- **LOW, `internal/adoptcli/adopt.go:231`.** The line `... was left unmoved: run lym fabric pull in %s` still names `worktree`, which is the root. The adopt "unmoved branch" notice is a reconcile-stop way forward of the same kind F1 and F3 convert. The fix, and doc.go's new narrowed sentence ("a reconcile stop"), leave it out. The sentence in `internal/adoptcli/doc.go:300` likewise names the branch's worktree. I did not read what `worktree` is at that site, so it may already be the anchored directory; if it is not, convert it or accept it as outside the issue.

No other unconverted member in the diff's set:
- F1: `pull.go:461` and `anchorstop.go:170` now go through `lymVerbDir`.
- F2, F4: the test rows were not searched, since their set is the changed lines themselves.
- F3: no other `BranchRewritten` way-forward site remains.
- F5: the remaining root-path `lym fabric checkout` refusals in `addparent.go`, `addadoptonly.go` and `battencli/readoptstop.go` are the ones the review said to leave unchanged.

Tool calls:
1. `grep` over non-test Go sources for `lym fabric` verbs followed by `in %s` without `lymVerbDir`. My context lacked any repo-wide search of unconverted sites.
