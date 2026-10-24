```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F2–F6 change set for consistency with its siblings.

**Findings**

1. **LOW** — `internal/gatecli/doc.go:126` and `:43`
   - The pruning and diff-base paragraphs are stale against the new behaviour.
   - Line 126 says each write keeps "each fingerprint's 20 newest whole-green commits".
     `green.go` now counts only whole entries with empty args against that budget.
     It also drops args-carrying whole entries of other trees.
     The `recordGreen` comment was updated, but the module doc was not.
   - Line 43 says "the newest commit that is an ancestor of HEAD".
     The code now picks the newest by history and falls back to write time only for unrelated commits.
     That agrees with the doc, so line 43 is not a finding.
   - Because of line 126, the module doc and the code disagree on the budget.

2. **LOW** — `internal/gatecli/doc.go:200` (F4 twin)
   - The example is fixed for `tier1` and `tier2`.
     `tier1` now compiles `tier1,integration`, and `tier2` now compiles `tier2,tmux`.
   - `internal/loomshed/verifygate_integration_test.go:131` and `:138` still show `-tags integration` and `-tags tmux` as tier commands.
     They are test fixtures, not docs.
     I did not check whether they run the empty-tier check.
     If they do, the old tags would trip it.
     Treat this as unverified.

3. **LOW** — `internal/gatecli/green_test.go`, `TestGreenRecord_NewestWholeAncestor`
   - The two new history rows both use `linear("old","new")`.
     Entry order is the only difference between them.
     The row name "wins over a later write of an older commit" is inaccurate.
   - The first row has `whole("new",0)` and `whole("old",time.Hour)`, so "old" really is written later.
     The second row is just the reverse order.
     The names are fine for that.
   - No row covers two history-unrelated commits where the older write wins the tiebreak against iteration order.
     The first row, "by write time among unrelated commits", covers that.
   - No finding.

Nothing else disagrees: the `isAncestor` seam has one other caller (`scope.go`), updated in the same change, and `green_test.go` was updated with it. The `"HEAD"` literal at both call sites is consistent.

Tool calls:
1. `grep` of `doc.go` for pruning, newest and tags: to see which doc paragraphs describe the changed behaviour.
2. `grep -rn` for old tag spellings across the repo: to find twins of the F4 doc fix.
