```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I name the test that pins it, or none.

**Per behaviour, the test that pins it**
- The sweep paragraph added to `contracts/stencils/webster/webster-body-implementer.md` (lines 55-58) is static stencil prose, so it needs no behaviour test.
- The renamed case title in `TestRenderPrompts` (`internal/websterengine/template_test.go:801`) changes no behaviour. The case still runs `renderFork` and checks its output.
- The two new `contains` strings (`template_test.go:810-811`) pin only static stencil prose, so they pin no behaviour. They are the finding below.
- The diff adds no code, so no behaviour is left uncovered.

**Findings**

- **F-T1, LOW, class `consistency`, location `internal/websterengine/template_test.go:810-811`.**
  - Summary: the two added `contains` assertions check static stencil prose, which the rubric's test-economy rule flags: a new assertion on a stencil's static prose, naming the assertion.
  - The two strings are `"   Grep the whole worktree for every retired name in one Bash call, test files, `testdata/` and other fixtures included"` and `"   Read every hit's span in one message, then edit every hit in one message"`.
  - Neither string pins a rendered or conditional behaviour. The prose sits outside any `{{...}}` template action, so the render cannot change it, and a reword of the paragraph breaks them for no behavioural reason.
  - The sibling assertion at `template_test.go:809` (`Locate them with one lym quarry resolve ...`) already exists. It pins only that the fork stencil renders, which makes the new strings redundant, not coverage.
  - Fix: delete both added `contains` lines at `template_test.go:810-811` and restore the case title at line 801 to its original wording.

**Tool calls:** none. My inherited context already held the diff and the test excerpt.
