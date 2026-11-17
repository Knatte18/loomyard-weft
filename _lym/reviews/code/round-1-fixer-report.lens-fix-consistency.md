```yaml
lens: fix-consistency
outcome: reported
```

Task: check the change to `_lym/landing/summary.md` for agreement with commit `382a591f` and the rest of the file.

**Finding 1 — LOW**
- Changed text: third bullet in `_lym/landing/summary.md`: "The `TestRenderOverview` doc in `internal/heddleengine/overview_test.go` broke two blocks after a comma mid-sentence."
- Summary: the bullet misdescribes the commit. The `TestRenderOverview` doc had one sentence broken after a comma, not two blocks.
- Evidence: `git show 382a591f` changes `overview_test.go` in one hunk. That hunk turns one sentence, "It pins the settings object with its setters marked, a note for the entry that holds no object," + "a place no entry reaches left out, and a second call changing nothing.", into two sentences.
- The reviewer's fix text said "whose two blocks broke after a comma". It most likely meant the two comment blocks, `overviewPlace` and `TestRenderOverview`. The fixer carried that phrase over as a count on the test doc alone.
- Suggested wording: "The `TestRenderOverview` doc in `internal/heddleengine/overview_test.go` broke a sentence after a comma mid-sentence."

The rest agrees with the commit:
- The header of `overview.go` was one 124-character line, and the commit splits it.
- The `overviewPlace` doc broke after a comma, and the commit splits it.
- The summary's "one commit, `382a591f`" matches the log.
- "Comment wording only, code untouched" matches the diff.
- Nothing else in `summary.md` refers to the old "no commit after it" or "no other edit" claims.

Calls made:
1. `git -C … show 382a591f`: to read the commit's diff, which my context lacked.
