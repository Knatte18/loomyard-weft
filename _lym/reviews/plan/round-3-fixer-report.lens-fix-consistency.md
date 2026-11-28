```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency: whether the round-3 change set agrees with the sibling cards and with the rest of card 12.

1. LOW — card 12 `**Uses:**` list against the rewritten support bullets (the `teardownRecorder` bullet, the hub-building bullet and the `TestReconcileOrigin_OwnedElsewhere` description).
   Summary: the new text calls exports that the Uses list does not name. The Uses list still names the other exports the card calls, so it is now out of step with the prose.
   Evidence: the changed text uses `pairteardown.New`, `pairteardown.HubSeamOptions`, `OriginSeams`, `pairteardown.OwnerReader`, `lymcwd.ResolveWorktree` and `gitkit.Git`. The Uses list does name `internal/gitkit#RevParse`, `#BranchExists`, `#CommitFile`, `internal/hubforge#CopyHub` and `#AddPair`, but it has no `internal/pairteardown#…`, `internal/lymcwd#ResolveWorktree` or `internal/gitkit#Git` line.

2. LOW — card 12 `**Extends:**`, which gains `internal/gitkit/doc.go`, against the closing paragraph.
   Summary: the new Extends target has no matching sentence saying what that doc addition holds.
   Evidence: the closing paragraph still reads "The boardengine doc addition says the package's tests hold fabric's flows that rest on the board's replay." Nothing in the card says what `gitkit/doc.go` gains for `CommitOnBare`.

Noted, out of scope because the fix only re-split this sentence: card 10 line 65 says card 12 deletes the run-scope and earlier-snapshot-tag rows as held by `TestEndedTeardowns`. Card 12 still lists "the run-scope rows" and "It keeps the earlier-snapshot-tag row". That disagreement predates this change.

Tool calls:
- grep of `_lym/plan/0*.md` and `1*.md` for the removed and renamed helper names, and of card 12's Uses block. My context held only cards 9 and 12, not the other cards.
- grep of `_lym/plan/10-origin-ended-core.md` for `TestEndedTeardowns`. Card 10 was not in my context, and the changed text points to it.
