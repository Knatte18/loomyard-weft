```yaml
lens: fix-consistency
outcome: reported
```

No findings.

Lens checked: the change set agrees with its surroundings.
- The sandbox doc line now says `wide`, matching `template.yaml`'s `active: "wide"`. A repo-wide grep for `cautious` next to "default" or "active", excluding tests, research docs and the template, finds no other place that still names `cautious` as the default. The only hit is the intended `formerTemplateValues` entry at `internal/configsync/configsync.go:53`.
- The two new `TestReconcileAll_MigratesToSparse` rows follow the table's existing conventions. The stripped row sets `resolutionChanges` because `cautious` resolves to `wide` after the strip. The marked row uses `marked: true` with `wantFile`, like the existing "marked worktree" rows.
- The new batcher subtest asserts the budgets 300000 and 200000 and equal `max_cards` and `batchifier`. These match the template text I saw (`budget: 300000` for `wide`, `budget: 200000` for `cautious`, `max_cards: 6` in both).
- The subtest decodes the template directly because no accessor exposes budget or `max_cards`, which is the review's stated fallback.

Tool calls made:
1. `git grep` for `cautious` near "default" or "active" across the repo, minus tests, research and the template: my context held no repo-wide search of other places naming `cautious` as the default.
