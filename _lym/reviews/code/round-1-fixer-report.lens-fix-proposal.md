```yaml
lens: fix-proposal
outcome: reported
```

fix-proposal: I challenged the changes carried out for review findings F1, F2 and F3, all fixed as proposed.

**LOW**: `internal/gatecli/scopes_integration_test.go:66`, the `scopesHub.plan` field comment ("the one-card plan written under the pair's never-tracked plan directory")

- **Summary:** F2's fix corrected the "never-tracked" wording in `_lym/landing/summary.md` but left the same claim about the same directory in the field comment this change added.
- **Evidence:** `git log -S"never-tracked plan directory"` shows `c5a7ccb1e`, a commit of this change, introduced the comment. The F2 review block says `_lym` "is the tracked records namespace, not a never-tracked one". The corrected summary now says "`_lym/plan`, which the pair's code repository excludes", so the two disagree. The wording that holds is that the pair's code repository excludes the directory.

I found no case against the F1 or F3 proposals:

- **F1:** the `--all` run now records green at the `other/other.go` commit, which differs from `f.mergeBase`. A gate that wrongly used that ancestor on a dirty tree would print a different commit.
- **F3:** the rows that build no hub are the ones that do not run in a hub.

Tool calls:
1. `git log -S"never-tracked plan directory"` on the scopes test file: my context did not show whether the field comment predated this change.
