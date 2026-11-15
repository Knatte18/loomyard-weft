```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. List, per behaviour the diff adds or changes, the test that pins it.

**Per-behaviour list**

- `edit-directive.md` gains three sentences: the link note, the relative-reference note and the absolute-path instruction. This is static stencil prose, which the lens exempts. Existing tests still cover how the directive renders:
  - `TestDirective_RendersTheRuleWithoutBannerOrMarkers` (`internal/editdirective/editdirective_test.go:16`) checks the anchor phrase, no `{{`, no leading `<!--` and no trailing newline. The new lines add no marker and no leading comment, so it still holds.
  - `TestStencils_NoEditRuleRestatement` (`contracts/stencils/parentdirective_test.go:150`) skips the `edit-directive` stencil, so it can't be affected by the new text.
- `internal/editdirective/doc.go` and `internal/fabricengine/doc.go` gain doc-comment text only. They change no behaviour, so no test applies.
- The diff adds or changes no Go code, signature, output or file shape, so no behaviour is left without a covering test.

**Findings:** none.

- The diff adds no test, so there is no duplicate-test, per-helper or prose-assertion finding to raise.
- The summary's claim that "no test pins the new sentences" matches the diff.

**Tool calls:** none. My inherited context already held everything this lens needed.
