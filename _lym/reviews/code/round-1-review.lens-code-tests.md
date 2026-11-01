```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I name the test that pins it, or none.

**Per-behaviour list**

1. New stencil prose in `contracts/stencils/burler/burler-fork-reading.md:19-26` (the waves paragraph). It is static stencil text, so it is exempt. No test pins it and none is needed.
2. Rendering of the fork-reading stencil. The added paragraph declares no marker and adds no placeholder.
   - `contracts/stencils/markers_test.go:29` pins `burler-fork-reading` with no markers. The added text introduces none, so the row still holds.
   - `internal/burlerengine/template_test.go:76` and `:184` pin the `## Reading for the fork reviewers` heading and the placement of the fork_reading marker. The diff does not touch the heading or the placement, so both rows still hold.
   - I did not run these tests. Passing is the verify gate's job.
3. The new sentence in `internal/burlerengine/doc.go:286` is a comment only. It has no behaviour and needs no test.

**Findings:** none.
- No behaviour is added without coverage.
- No new assertion on static prose is present or needed.
- No existing stencil-rendering test is broken by the added text. Its heading, marker and placement anchors are unchanged.

**Tool calls:** none. The grep results already in my context showed `markers_test.go:29`, `template_test.go:76` and `template_test.go:184`, and I made no further reads.
