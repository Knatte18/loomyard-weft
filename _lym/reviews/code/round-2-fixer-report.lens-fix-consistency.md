```yaml
lens: fix-consistency
outcome: reported
```

No findings.

- **Row placement and name:** the row sits after `EmbedPackageDoesNotParse`, as F1 asked. It follows the other rows' `committed`, `files` and `inert` conventions.
- **Fixture:** it replaces `a/assets.go` with `package a\n`, so the `//go:embed static` directive no longer names `page.txt` (`testselect_test.go:74`). It also replaces `a/a_test.go` (line 77) with a `go:embed static/page.txt` directive.
- **Map:** `committed` is applied before `selectFixtureMap(committed)` builds the map. The map's `Linked["m/a"]` hashes therefore match the working tree, apart from the `a/static/page.txt` change in `pageChanged`.
- **Other fixtures:** `var page embed.FS` does not clash with any other declaration in package `a` (`linkedSource` or `gone.go`). The new row does not touch `committedFiles` or the sibling rows' fixtures.
- **Expected outcome:** the row sets no `wantMiss` and no `want`, so it expects no miss and no tests. This matches the review's description (current code selects nothing; it would miss if linked test files were read for embeds).
- **Twin rows:** `OwnTestFileEmbedsInertFile` pins the own-package half, and this row pins the linked half. Nothing else in the change set has a twin left stale.

Tool calls made:
1. Read `testselect_test.go` lines 60-165, to check `committedFiles`, `embedAssets`, `pageChanged` and `selectFixtureMap` against the new row. My context held only the later span of the file.
