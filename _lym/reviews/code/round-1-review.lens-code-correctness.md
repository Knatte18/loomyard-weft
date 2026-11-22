```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the diff's changed functions and branches (tests only).

**Functions and branches, the case run and the result:**
- `TestRenderBoardData/EntriesStandInTheirProjectAndSection`, the count check: the fixture has 16 tasks, 4 of them projects (P1–P4), so 12 tasks and notes. `want` has 12 keys, so `len(want)` matches the old `len(tasks)-4`. Correct.
- The same subtest's unknown-handle branch: an entry whose handle is missing from `want` reports an error and continues instead of indexing a zero `placement`. Correct.
- The same subtest's `waitsOn` compare: an entry with no waits gets nil from the JSON decode, and `want` has nil. `slices.Equal(nil, nil)` and `slices.Equal(nil, []string{})` are both true, so no false failure. T3 `["T2","ghost"]` keeps declaration order, the same order the old README After cell gave. Correct.
- The same subtest, T10 (`gtask` under done project P3): the old test asserted only `project == "P3"` and skipped `section`. The new row also pins `section: "Ready"`. That is stricter, not looser, and the darn-write gate's reports match HEAD's tree, which indicates it passed. Correct.
- `HeaderAndSections`: the sections literal replaces `rootSectionNames`. The old code skipped the Tasks and Notes headings and took the rest in README order. The literal lists every subsection a root holding one entry of each kind shows: Running, Ready, Layer A, Layer B, Independent, Draft, Bugs, Enhancements, Done. That matches the fixture comment and the projection order. Correct.
- `RenderToDiskClearsAMissingDecisionRecord…`, the new `given[0].Discussed` assertion: `RenderToDisk` passes `withMissingRecordsUndiscussed(boardPath, tasks)`, which works on a `slices.Clone` (`internal/boardengine/render.go`, `withMissingRecordsUndiscussed`). The caller's slice therefore keeps `Discussed == true`, the assertion holds, and a mutation of the caller's slice would make it fail. Correct, and it carries over the deleted test's check unchanged in meaning.
- `TestRenderToDiskManifestCleanup/SeededManifestRemovesOrphanAndWritesOutputs`, the removed `BoardDataFile` stat: the loop over `BoardDataFile` and `BoardPageFile` that follows still stats both files and checks each in the manifest (`internal/boardengine/render_test.go:424-431` at HEAD). Nothing is lost. Correct.
- The same subtest's end-of-line comment moved onto its own line: no behavior change. Correct.
- `TestRenderOrder`'s `//testtiming:keep` reason, a comment only: no behavior. Correct.
- The deleted test files and helpers: nothing on the production side referenced them. The remaining `render_test.go` imports (`encoding/json`, `os`, `path/filepath`, `strings`, `testing`, `time`) are all still used, and `view_test.go`'s dropped `regexp` import has no other user. Correct.

**Findings:** none. The diff changes no production function, and every changed test branch computes what its message states on the inputs above.

**Tool calls:** none. My context already held every definition the procedure needed.
