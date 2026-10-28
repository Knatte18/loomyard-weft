```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I name the test that would fail if it broke.

**Per behaviour, the test that pins it**
- **Order and separator, no deny notice:** `TestPrepare_AppendSystemPromptMatchesDenyNotice` in `prepare_test.go` (about line 325). When `notice == ""` it requires the file to equal `creelTrustSentence + " " + batchingSentence` exactly, so a missing sentence, a wrong order or a wrong separator fails it.
- **Order and separator, with a deny notice:** the same test, whose `want = notice + " " + want` branch covers every deny combination. `TestPrepare_WritesArtifactsAndReturnsConsistentLaunch` in `settings_test.go:750` repeats the check for one concrete notice. Both use exact equality. The new behaviour needed no new test function, only edited expectations, so there is no duplicate coverage.
- **Launch and resume lines name the file:** the existing assertions in `TestPrepare_AppendSystemPromptMatchesDenyNotice` and the `command.go` tests cover this. The diff changes only a comment there.
- **`init` refuses a line break in the batching sentence:** no test covers it. A package-load panic over a constant cannot be tested without editing the constant, and the creel check it extends had no test either. This is not a finding.
- **Batching sentence text equals the board entry's sentence:** the tests compare against the `batchingSentence` constant, so an edit to the constant's wording would not fail any test. This matches how the creel sentence is tested. I raise it as a LOW below.
- **The eight stencil sentences:** static prose, exempt. The marker positions are pinned by the existing `contracts/stencils/markers_test.go`, which the diff does not touch.
- **`docs/overview.md`, `doc.go` and comment edits:** prose, exempt.

**Findings**

1. LOW, consistency, `internal/shuttleengine/claudeengine/prepare_test.go:293`. The doc comment of `TestPrepare_AppendSystemPromptMatchesDenyNotice` still says the file "holds the deny notice, a space and the creel trust sentence, or the sentence alone". The diff changed what the test asserts at line 325 but not this comment. Fix: reword it to "the deny notice, the creel trust sentence and the batching sentence, space-separated, with the notice left out when empty".

2. LOW, design, `internal/shuttleengine/claudeengine/prepare_test.go:325` and `settings_test.go:750`. The tests take the expected text from the `batchingSentence` constant, so they cannot catch the wording drifting from the board entry's required sentence. This matches how the creel sentence is tested. Fix, optional: pin the literal "Request every call whose arguments you already know in one message, and read what you need before you edit; wait for a result only to choose the next call." in one assertion. Otherwise leave it as is.

**Tool calls:** none. The manifest's diff and file reads already held everything this lens needed.
