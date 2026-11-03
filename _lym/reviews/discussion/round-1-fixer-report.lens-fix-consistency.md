```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked whether the changed decision record and support log still agree with their sibling decisions, criteria and cross-references.

- **MEDIUM.** Changed text: D6's new cleanup-sweep paragraph ("under the slug's teardown lock it reads the weft branch's origin tip, archives that tip as well … and leases the remote delete on it").
  Summary: the sibling D7 bullet and its acceptance criterion were left as they were, so they disagree with D6 about when the sweep holds the lock.
  Evidence: D7 still says "The cleanup sweep takes it with one try per slug before that slug's remote deletes; a held lock skips the slug". The D7 criterion still says the sweep "skips that slug's remote deletes". In `Cleanup` (`internal/fabricengine/cleanup.go`), `archiveWeftTip` and the local `deleteWeftBranch` run before `deleteWeftBranchOnRemote`. So the read and archive of the origin tip that D6 puts under the lock happen before the point where D7 takes it. D7 should take the lock before the slug's archive. A held lock should then skip the whole slug, not only its remote deletes, or the sweep archives and deletes locally while a removal or `Add` holds the slug.

- **LOW.** Changed text: D3 now names `lym batten cancel` only before Land, and at Land it names D2's Land way forward instead.
  Summary: the twin of this decision in the support log's question ledger was left stale.
  Evidence: Q5 in `support-log.md` still reads "Name `lym batten cancel` in the way forward and in fabric remove's warning (D3)". It does not say that this applies only before Land, and it does not mention the Finalize exception. The fixer did update Q8 for the same round, so Q5 stands out.

No further tool calls were made.
