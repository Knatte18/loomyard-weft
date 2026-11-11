```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the F1 proposal, the new `LinkedTestFileEmbedIsNoSource` row, and check it fails with the defect in place. I found no findings.

- **Fails with the defect in place:** the row's `a/a_test.go` embeds `static/page.txt`. `linkedChanges` calls `otherFileMiss(dir, rel, false)` for `a/static/page.txt`, and `embedPatterns` skips `_test.go` files when `withTests` is false. If a linked package were read with `withTests` true, `ownedFiles("a")` would include `a/a_test.go` and `static/page.txt` would match, giving a `MissNonGoFile`. The row expects no miss, so it would fail.
- **Passes for the intended reason:** the row replaces `a/assets.go` with `package a\n`, so the committed `//go:embed static` directive can no longer name `page.txt`. `a/a_test.go` is in `committedFiles`, so the override replaces it in both the committed and the working tree. `pageChanged` and `inertText` make the change an inert non-Go file, so the only possible miss path is the embed check. The row does not trip other paths: a linked `_test.go` is not edited, and `a/assets.go` is unchanged between the committed and working trees.
- **Placement and shape:** it follows `EmbedPackageDoesNotParse` as proposed and uses the existing `committed`, `files` and `inert` fields. The `wantMiss` and `want` zero values are handled by the existing switch.
- **Overlap with other rows:** it duplicates none of them. `OwnTestFileEmbedsInertFile` pins the own-package half, and no other row covers the linked half.
- **Test run:** the fixer's `lym gate test` run passed.

Further tool calls (1): one `Bash` call read `testselect.go` (`linkedChanges`, `embedPatterns`) and the fixture and helper definitions in `testselect_test.go` (`committedFiles`, `embedAssets`, `pageChanged`). The context held the test table only, not the production code or the fixture.
