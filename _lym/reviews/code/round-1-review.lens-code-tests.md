```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens — for each behaviour the diff changes, name the test that pins it.

**Per-behaviour list**

- The interactive branch of `raddleModeRules` now asks for recipe, profile and dependency columns and limits the surrounding prose. This is prompt text, static prose, so it is exempt from the coverage rule. The only existing pin is `TestRaddleTable` (`internal/loomengine/raddletable_test.go:88`). It builds its expected string by calling `raddleModeRules` itself, so it passes for any wording. It does not fail if the columns are dropped, and I found no reason to want it to.
- The diff changes no signature, return shape or non-text behaviour, so no behaviour is left uncovered.
- The diff adds no tests. That is correct under test economy: a new assertion on static prose would itself be a finding.
- No existing assertion on the old table wording goes stale. A grep of `raddletable_test.go` shows no assertion containing "dependencies" or "table" text. Line 88 compares against the function's own output, and line 101 checks that other modes' text is absent.

**Findings:** none.

**Tool calls:** none. The manifest already held `raddletable_test.go` greps and the diff.
