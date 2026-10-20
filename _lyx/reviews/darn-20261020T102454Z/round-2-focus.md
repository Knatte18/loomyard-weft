---
round: 2
focus:
  - "Does every verify command the diff derives or documents, including the impacted-set command built in `buildCommand` in `internal/impactset/derive.go`, run the `integration` tier alone, with no untagged `go test` step before it, as the board entry asks?"
  - "In `storePrebuilt` in `internal/gatecli/prebuild.go`, does a prune that removes an entry after the `os.Chtimes` refresh leave the run exporting a path that exists, and does a test reach the not-exist branch between the existence check and the refresh?"
---

Both entries name where to look and the question to settle; neither pre-states a verdict.
