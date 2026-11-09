```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fixer's change set for agreement with its surroundings (lens fix-consistency).

**Findings**

1. **MEDIUM** — `cmd/testtiming/main.go:68`, the `-redundancy` flag doc: "The header and every unmeasured section stay byte-identical". The F2 fix makes `spliceReport` (`cmd/testtiming/splice.go`) rewrite the header from `renderReport(tags, nil)`, so the sentence is now false. The same F2 change updated the `spliceReport` doc comment to say the header is rewritten, so the twin comment was left behind. Line 67 ("only the measured packages' sections change") is stale in the same way.

2. **LOW** — `cmd/testtiming/redundancy.go:194`, the `runRedundancy` doc: "Any other run rewrites only its own packages' sections of the existing one." It leaves out the header rewrite that F2 added.

3. **LOW** — `pattern/PATTERN-test-speed.md:248-249`: "A run narrowed with `-pkg` rewrites only its own packages' sections of the existing report". Same stale claim. The F1 edit sits directly beneath it and left it as it was.

**Checked and consistent**
- `cmd/testtiming/main.go:78` and the test comment at `cmd/testtiming/redundancy_test.go:23` already say "covers nothing of its package", so the F1 wording matches them.
- The `renderReport` header text (`cmd/testtiming/redundancy.go:171-176`) agrees with the F1 PATTERN sentence.
- `splitReport` still ends the header in one newline, and `renderReport(tags, nil)` produces the same shape, so the splice output is well formed.

**Calls made**
- `grep` for the "only its own packages' sections / byte-identical / rewrites only / covers nothing" phrases: my context lacked the other docs that restate the splice contract.
- `sed -n 62,72p cmd/testtiming/main.go`: the `-redundancy` flag doc was not in my context.
