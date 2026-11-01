```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, which lists each behaviour the diff adds or changes and the test that pins it.

**Per behaviour, the test that pins it**
- The diff changes `PATTERN.md`, `pattern/PATTERN-fabric-isolation.md` and `pattern/PATTERN-no-hard-coded-choice.md` only. It has no Go code, stencil logic or runtime behaviour, so there is nothing for a test row to pin.
- The six new entries carry no `(test)` marker. The summary says review holds them and gives that as a design decision, so there is no claimed test to find missing.
- The only mechanical check on the diff is `lym loom lint-pattern` (the summary says it passes). It is a gate, so a finding about it would duplicate the verify gate's work. I did not re-run it.

**Findings:** none.

**Tool calls:** 0. The context already held the full diff, the commit message and the summary.
