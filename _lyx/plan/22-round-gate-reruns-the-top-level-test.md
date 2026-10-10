# Card 22 — round-gate-reruns-the-top-level-test

**Edit:**
- `internal/loomshed#publishFailureCommand`
- `internal/loomshed#anchoredRunPattern`
- `internal/loomshed/publishfailure_test.go`

**Intent:** Have the round gate's failing-test re-run name the top-level test only (D10).
`anchoredRunPattern` anchors the first `/` segment alone (`^TestX$`), and `publishFailureCommand` collapses the record's tests to one step per package and top-level test, keeping the record's order of first appearance; `PublishFailureNote` keeps the full subtest path.
`TestPublishFailure`'s command subtest gains the acceptance row: `TestA/sub_1` and `TestA/sub_2` in one package emit one `-run '^TestA$'` step, and `TestPublishFailureNote` keeps both subtest paths.

**ImpactSummary:** Two unexported helpers in loomshed; the verify gate that joins the command is unchanged.
