```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency lens on the round-1 change set, looking for twins left unchanged.

- **LOW** — `internal/gateslot/doc.go:252-253`, the `TierRunFailures` summary in the package doc.
  - The summary says `TierRunFailures` "reports false unless every run the report lists is a tier or guard run."
  - The fix added a second false case, a package that failed outside any test. The `runlog.go` doc comment and the `landingshed` package doc (`doc.go:252`) both name it.
  - The `gateslot` package doc was left as it was, so it now understates the false cases.
  - Evidence: `runlog.go:366` ("So does a package that failed outside any test…"), while `gateslot/doc.go:253` has only the run-type condition.
  - Suggested fix: add "or a package failed outside any test" to the second sentence.

I found no other stale twins:
- **Renamed constants:** no use of `reportCutPrefix` or `reportOutputLine` is left outside tests.
- **`gatecli/doc.go:605`:** it describes how the report renders a package that failed outside any test. That output is unchanged, so it still agrees.
- **`loomshed` integration test (`outsideAnyTest`):** it exercises `FailedTests`, not `TierRunFailures`, so the new false case doesn't touch it.

Tool calls:
1. `grep` across `.go` and `.md` for `TierRunFailures`, `reportCutPrefix`, `reportOutputLine` and `failed outside any test` outside tests — to find twins my context lacked.
2. `sed -n 244,262p internal/gateslot/doc.go` — my context lacked the surrounding text of the `TierRunFailures` mention in the package doc.
