```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the F1 and F2 test additions through the correctness lens.

No findings.

- **F2 row (`rubric_test.go`):** a reader returning `" "` reaches `noteOrNone` and renders `Redundancy: none`. The row fails if the blank-answer handling at `rubric.go:90` is dropped, because the output would then be `Redundancy:  ` with a space. The tier0 gate for `./internal/shedadapters` passed.
- **F1 malformed row:** `{not json` makes `gatecli.ReadRedundancyReport` fail, and the closure returns the "the report could not be read: ..." line. The `HasPrefix` check fails if that branch is removed or reworded.
- **F1 matching row:**
  - The closure reads the report, and `fabricengine.Open` and `HeadSHA` give the pair's HEAD.
  - `RedundancyReportMatches` finds the parent merge base, which is HEAD itself for a fresh pair, so the changed set and the selected tests are empty.
  - That equals the report's empty `TestHashes`, and the closure returns the "matches HEAD's tree" line and the test lines.
  - If the HEAD wiring broke, for example by passing an empty rev, `ChangedPathsBothSides` would error and the row would get the "could not be matched" line instead.
  - The tier1 run of `./internal/loomcli` passed, and its output names `ReadsTheWorktreeReport`.
- **Caller check:** no signature or behaviour changed outside the tests, so no caller can break.

Tool calls: none; my context already held every changed file and the gate results.
