```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I name the test that pins it, or none.

**Per behaviour**

1. **The new `batchingSentence` text, as it reaches `append-system-prompt.txt`.**
   - Pinned by `settings_test.go:803` and `prepare_test.go:398`.
   - Both build the expectation from the constant, so they check delivery and ordering: the sentence follows the trust sentence, the deny notice comes first when present, and the same file is used on the launch and resume lines.
   - Neither test fails on a wording change, and none should.
   - A test that asserted the wording would be a new assertion on static prose, which test economy forbids.
   - Not a finding.

2. **The one-line guarantee for the longer sentence.**
   - Pinned by the `init()` loop at `settings.go:461`, which checks `\n` and `\r` for both sentences. A violation panics at package load, so every test in the package fails.
   - The colon and the semicolon are not in the forbidden set, so the new text passes.
   - Not a finding.

3. **Space-joinability with `creelTrustSentence`.**
   - Pinned by the same two tests, which compare against `creelTrustSentence + " " + batchingSentence`.
   - `TestPrepare_AppendsStandingPassages` (`prepare_test.go:520`) also pins the sentences-first ordering before the standing passages.
   - Not a finding.

4. **The `doc.go` paragraph.**
   - It is prose with no test, which the lens exempts.
   - None needed.

5. **The Webster implementer stencil rewrite.**
   - It is static prose, which the lens exempts.
   - Not a finding.
   - I did not check whether a stencil render test pins the stencil. My manifest has no such test, and a wording-pinning assertion would itself be a finding.

**Findings:** none.

**Tool calls (1 of 6)**
- `grep -rn "batching|batchingSentence|Request every call" internal/shuttleengine/claudeengine/*_test.go`. My context did not show whether any test asserts the sentence's literal wording, or which tests build their expectation from the constant.
