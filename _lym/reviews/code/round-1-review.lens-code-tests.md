```yaml
lens: code-tests
outcome: reported
```

**Lens: code-tests.** For each behaviour the diff adds or changes, the test that pins it:

- **Marker parsing, stacked `//lym:watch` lines, and a misplaced or malformed marker refusing the gate.** `internal/impactset/watches_test.go:30-72` pins these. This is existing behaviour and the diff adds no new parsing. No finding.
- **Skipping a watched guard when no changed file matches, and running it when its own file changes.** `UnwatchedTests` and the gate's `-skip` are covered by the existing impactset and gatecli tests, which the doc at `internal/gatecli/doc.go:323-340` describes. The diff does not change the behaviour. No finding.
- **The 25 new pattern sets covering every file their guard reads.** No test pins this. It is declarative metadata, so a test that mirrored each pattern list would only restate the markers. I raise no finding, because the diff adds no executable behaviour here. A stale or over-narrow pattern is a review judgement, which the handler and the other lenses cover.
- **Each guard's own outcome.** The diff does not change any test body, so the tests that already exercise each guard still pin it.
- **The `PATTERN-test-speed.md` bullets.** These are prose and exempt.
- **The added doc-comment sentences.** These are prose and exempt.
- **Test economy.** The diff adds no test function, so there is no duplicate-test or per-helper-test finding.

Findings: none.

Tool calls (1 of 6 used):
1. `grep -rn "lym:watch" internal/impactset/*_test.go`: to confirm that marker parsing, stacking and misplacement already have test rows. My context held the gate doc but not the impactset tests.
