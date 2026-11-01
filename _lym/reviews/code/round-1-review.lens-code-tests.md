```yaml
lens: code-tests
outcome: reported
```

Task: list, per behaviour the diff adds or changes, the test that pins it (lens code-tests).

**Per-behaviour list**

1. Shipped entry `lym-plan-edits-before-gate` exists, sits after `lym-batching`, targets `claude-file` alone: pinned by `TestStore/ShippedEntriesArePinned`, `internal/heddleengine/store_test.go:169-193`. It checks the slug at index 1, the single `claude-file` target, and the layer length.
2. The entry's text is the four-step command plus the forbidden-pattern sentence: pinned by none. The row's `wantText` is `""`, so the text goes unchecked. The text is stencil-like static prose, so the exemption applies. No finding.
3. The entry renders into `CLAUDE.local.md`: covered by the existing render tests. `claudeFileWant` (`internal/heddleengine/render_test.go:73-89`) loops over every shipped entry targeted at `claude-file`, so the render tests follow the new entry without edits.
4. The `lym heddle list` output includes the new slug: covered by the existing heddlecli tests. `shippedSlugs()` (`internal/heddlecli/cli_integration_test.go:157-163`) derives the slugs from the layer.
5. Tests that index `Ordered()[0]` (`render_test.go:296,305,345,371`, `store_test.go:149,208,220`, `override_test.go:14`) still target `lym-batching`, so they stay consistent. Those tests cover override and shadow behaviour for one shipped slug, and the new entry adds no behaviour there.
6. The renamed `ShippedEntriesArePinned` replaces `ShippedBatchingEntryIsPinned`. It keeps the old assertions: batching slug, target and exact text. It adds the layer-length check and the order check for the new entry. It is a table with rows for both entries, so it duplicates no other test. The `t.Fatalf` on length mismatch guards the later indexing. No finding.
7. Stencil batching lines removed (`contracts/stencils/**`): static prose, exempt. No stencil assertion was added or removed. No finding.

**Findings:** none.

**Tool calls**
1. Read `render_test.go:70-99`: to check how render tests derive the expected `claude-file` text from the shipped layer.
2. Read `cli_integration_test.go:145-175`: to check that the list tests derive the shipped slugs rather than hard-coding them.
