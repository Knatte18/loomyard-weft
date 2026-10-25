```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens, listing per behaviour the test that pins it, or none.

**Per-behaviour list**

The diff adds three files: `report.md`, `claude-mid-turn-30.tsv` and `claude-sendmessage.tsv`. It has no Go code and no test file, and `go test` cannot reach any of it.

- Report prose (claims, figures, recommendation): no test, and none is due. The rubric's test-economy exemption covers static prose, and a new assertion on it would itself be a finding.
- The two raw TSV logs (data fixtures cited by the report): no test reads them, and none is due. Nothing in the repo consumes them as input.
- No production behaviour is added or changed. The prototype stayed under `.scratch/`, so no uncovered behaviour exists to name under the "behaviour with no coverage" bullet.
- Redundancy checks (duplicate test, per-helper test, review-fix test, assertion on static prose): the diff adds no tests, so none of them applies.

**Findings:** none.

**Tool calls:** none. The diff stat and file list in my inherited context were enough to show that the diff has no test files and no code.
