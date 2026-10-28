```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix change set (the commits `90a77516`, `c2523623` and `6bfb76dc`) for agreement with its surroundings.

no findings

- **F5 comments:** the `projectFilePrefix` comment in `render.go` and the `boardProjectPrefix` comment in `replay.go` each name the other side, and both constants still hold `"project-"`.
- **F3 and F4 goldens:** the `ghost` dependency adds `` `ghost` `` to T3's After cell, and the other rows that name T3 or its After cell are unchanged. The `project-tidy.md` golden matches the shape of the sibling `project-ship.md` and `project-inner.md` goldens, including the intro text, the "grouping" line and the Draft heading and meaning. The `projectFiles` list already contained `project-tidy.md`.
- **Doc comment on the golden test:** the sentence added before "A fifth row" matches the two fixture changes.
- **F1 tests:** the build tags (`tier0` for `boardoverview_test.go`, `tier1` for the replay row) match the headers and the neighbouring tests. The new Bolt row follows the pattern of its sibling row and shares its `pushBase` and `requireClean` helpers.
- **Twins:** `boardDesignSlug` and `isBoardPath` have no sibling name rule left uncovered by this change, and no other site repeats the `project-` literal that the fix touched.

Further tool calls: none, because my context already held every changed file.
