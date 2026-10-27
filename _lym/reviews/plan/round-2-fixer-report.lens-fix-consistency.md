```yaml
lens: fix-consistency
outcome: reported
```

Task: check that this round's changes agree with the rest of the plan and the decision record (fix-consistency lens).

1. **LOW.** Card 8, "The kept tidy" paragraph, sentences "When it reports false and the read failed with an error other than a decode failure…" and "When it reports false and the read succeeded, `keptTidy` takes the record's `Tidy`, nil included."
   - Summary: the rewrite says nothing about what a decode failure, or an absent record, does to `keptTidy`.
   - Evidence: the old text covered both with "replaces `keptTidy` with its `Tidy`, nil included". The new text still says the read treats "an undecodable record as absent with one Warn". But its branches name only a non-decode error and "the read succeeded". `state.ReadJSONNoMkdir` (`internal/state/state.go:184-195`) returns `(zero, false, nil)` for a missing file and `ErrDecode` for an undecodable one. A reader can't tell whether those cases set `keptTidy` to nil or keep the earlier value.
2. **LOW.** Card 8, new row "a record holding a `Tidy` at start whose `state.json` tick 1's hook replaces with a directory…"
   - Summary: this row doesn't override the harness default about the watch event, though its twin row does.
   - Evidence: line 50 says "the tick hook removes the whole worktree on tick 1 and sends one watch event". The F1 row overrides that explicitly: "in this row tick 1's hook sends no watch event". The new row reuses "the same `Deps.Sleep` wrapper", which sends its own value on `Watch`. It says `.git` stays present, which implies no removal, but it never says the hook skips the default watch event.
3. **LOW.** Card 8, line 50 harness description, next to the F1 row's new "with the harness's `stopAt` 3 so the tick seam's cancel cannot mask a second tick".
   - Summary: `stopAt` is set only in the two Sleep rows, while the other beam rows that need an exit after a tick still give no `stopAt`.
   - Evidence: the review gives the hazard: "if it were 2 the tick seam's cancel would end the loop on tick 2 before the post-tick check". Rows 54–55 remove the worktree during tick 2 and depend on the post-tick exit. The harness's tick seam cancels at `len(h.ticks) >= h.stopAt` (`loop_test.go:96-97`). Those rows give no `stopAt`, so the same masking can hit them.

Card 11 changes: no disagreement found. The decision record's open-risks bullet ("gets `adopt` with the same tiers through the reconcile"), the parent decision at line 207, the `MigrateConfig` bullet (`sparseSeed` itself stays unseeded, since `MigrateHubBase` runs over its output in `configsync`) and the overview all agree with the new absent-branch text and the new test row.

Tool calls:
- grep of the decision record, overview and sibling cards for kept tidy, read-error, `ReadJSONNoMkdir`, worktree-gone and adopt-seed text: my context held only cards 8 and 11 and two of the decision record's entries.
- overview adopt lines, the decision record's open risks, and card 7's harness and Sleep text: to check sibling statements on the adopt seed, and the harness default the new rows override.
