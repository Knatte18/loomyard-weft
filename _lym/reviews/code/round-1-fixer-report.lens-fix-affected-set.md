```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Unchecked: F1. It is a single test row, not a set.

F2's premise is a set: every doc site that states the compile-check steer. I reran its search with `grep -rnoiE` for "package-scoped compile check" and "compile check" across `*.md` and `doc.go`, excluding `_lym`. It found two sites, and both state the `refuse_raw` condition:
- `internal/shellguard/doc.go:19` already says "each only while no refuse-raw prefix refuses it".
- `docs/overview.md:419` is the line the fix changed.

The grep only matched text with 80 characters of context on both sides, so a site near the start or end of a line would not match. The review also named `internal/shuttleengine/claudeengine/doc.go` as stating the condition. That file did not appear in the grep output, probably for that reason. I did not read it.

Tool calls (1):
1. `grep` for "compile check" in `*.md` and `doc.go`. My context held only the changed overview line, not the other sites.
