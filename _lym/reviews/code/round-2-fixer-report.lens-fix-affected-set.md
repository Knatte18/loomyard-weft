```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

The lens is the sets the fix had to cover. F1's set is the sites where a refusal is built together with a record write. That set has one member, and the new test covers it.

- The only refusal that comes with a write is the past-deadline branch at `internal/burlerengine/parentreview.go:224-227`.
- The other three refusals (lines 214, 217, 220) return `errParentAskUnchanged`, so nothing is written for them.
- The only non-test consumer of the past-deadline kind is `internal/loomcli/parentreview.go:170`. That is a caller, not a second write site.
- The fix is a new test, `TestAnswerParentAsk_FailedWriteAtDeadline`, which covers the one branch. Production code is untouched.

Unchecked findings: none.

Tool calls:
1. `grep` for `ParentAnswerPastDeadline` and `ParentAnswerRefusal{` across the repo's Go files. My context lacked a repo-wide list of refusal construction sites and consumers.
