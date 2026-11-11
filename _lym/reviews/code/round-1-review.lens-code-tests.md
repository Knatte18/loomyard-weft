```yaml
lens: code-tests
outcome: reported
```

**Per-behaviour list (lens code-tests)**

1. Engine: a Sketch promotion sets `depends_on` in the same write that releases it. Pinned by `TestDiscussedPromotion` row `PromoteOfASketchReleasesItWithItsDependsOn`, through the new `wantDependsOn` assertion (`internal/boardengine/sketch_test.go`).
2. Engine: a differing non-nil `depends_on` on an entry that is already a task is refused, naming `set-deps`. Pinned by the row `DependsOnOfAnEntryAlreadyATaskIsRefused`.
3. Engine: an already-task entry with an equal `depends_on` returns unchanged without a refusal (the `slices.Equal` branch, `store.go:805`). **None.** No row passes a `dependsOn` equal to the task's own, so inverting that condition would break no test.
4. Engine: an empty non-nil `depends_on` (`[]`) on an already-task entry that has dependencies is refused. **None.** This is the case where a null or `[]` payload is meant to differ from the task's own list.
5. CLI: `promote` accepts `depends_on` and the dependencies reach a plain note's task. Pinned by `stepPromote` in `internal/boardcli/verbs_test.go` (the `s` slug depending on `q`).
6. CLI: `promote` refuses a `depends_on` that is not an array, and one naming an unknown dependency. Pinned by the two new `stepPromote_Refusals` rows.
7. CLI: `promote` refuses an unknown key and accepts only `depends_on` besides the selector (`promotePayload`). Pinned by the existing `tier is an unknown key` and `unknown key` rows, plus `TestPayloadUse_MatchesDeclaredKeys` for the Use-token match.
8. CLI: `depends_on` null on `promote` reads as an empty list (`dependsOnList(nil)`). **None.** No row sends `"depends_on":null` to `promote`.
9. CLI: `depends_on` elements that are not strings are refused on `promote`. **None** for `promote`. My grep found no existing test with "elements must be strings", so the same message in `set-deps` has no visible test either.
10. CLI and engine: `promote` returns sibling warnings, and its `notice` carries `siblingNotice(siblings)`. **None.** Nothing asserts that `PromoteTo` returns siblings, or that the CLI envelope's `notice` is non-empty after a `promote` whose new `depends_on` is shared with a labelled sibling. The existing notice tests (`cli_test.go:209-226`) cover `set-deps` and `upsert` only. The `help_test.go` change only matches the word "notice" in the help text, which is static prose.
11. `dependsOnList` refactor of `set-deps`: the `set-deps` null-clears-list and non-array behaviours. Not changed by the diff, and I could not confirm a test. Part of finding F2 below.
12. `Board.Promote` wrapper keeps its no-siblings signature. Pinned by the many existing `Promote` callers (`board_test.go`, `handles_test.go`).
13. Redundancy check of `TestDiscussedPromotion`: the two new rows add distinct behaviours (dependencies set, and refusal). They duplicate no existing row, so no test-economy finding.

**Findings**

- F1, MEDIUM, design. Location: `internal/boardcli/cli.go:209` and `internal/boardengine/board.go:443`. Summary: the new `promote` sibling-warning behaviour, which `doc.go` and `--help` now promise, has no test; neither the siblings from `PromoteTo` nor the `notice` field of the CLI envelope is asserted. Evidence: `board.go:443-458` and `cli.go:209-212`; `cli_test.go:209-226` covers only `set-deps` and `upsert`.
- F2, LOW, design. Location: `internal/boardcli/cli.go:515` (`dependsOnList`). Summary: the shared parser's null and `[]` handling for `promote`, and its non-string-element refusal, are unpinned. Add `promote` rows `{"slug":"p","depends_on":null}` and `{"slug":"p","depends_on":[1]}`. Also add an engine row for a task that already has dependencies promoted with `[]`. Evidence: `cli.go:515-536`; the `stepPromote_Refusals` rows in `verbs_test.go:102-112` hold neither case.
- F3, LOW, design. Location: `internal/boardengine/store.go:805`. Summary: the equal-`depends_on` already-task path, which must stay a silent no-op, has no row. Add an engine row promoting a task with a `dependsOn` equal to its own and expecting no error and no change. Evidence: the `TestDiscussedPromotion` rows touch only the differing case.

**Tool calls**
1. One grep of `internal/boardcli/*_test.go` for notice, sibling and array-error assertions, and of `PromoteTo(` test callers. My context held only the diff, not the existing `set-deps` and notice tests.
