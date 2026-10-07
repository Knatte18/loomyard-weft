---
round: 3
focus:
  - "In `internal/fabricengine/refscanner.go`, `IsReadOnlyCommand` splits a command into segments and tests each segment's first word against a closed reader list: settle whether a quoted or backslash-escaped command word (for example `'tee' cat`, `\"rm\" cat x`, `ls; 'git' cat`) can still be classified read-only, and whether `TestIsReadOnlyCommand` in `refscanner_test.go` has a row for it."
  - "In `internal/websterengine/recoverbatch.go`, the recover-batch refusal's way-forward text and the `audit: fabric-reference` row of `contracts/specs/refusal-spec.md`: settle whether the message names both the HEAD-at-start route and the committed read-only route as the row does, and whether `recoverbatch_test.go` pins that wording."
  - "In `internal/webstercli/cli.go`, the usage line for `accept-audit --batch` against the verb's `Long` help: settle whether both describe the same set of accepted batches."
  - "In `tools/tokencount` (`calibrate.go` growth ratio, `profile.go` `countFindings` and `readRunProfile`): settle whether the `n/a` growth ratio, a malformed review file and a no-partition run with an unreadable plan each have a row in `TestCalibrate` or `TestBuildProfileReport`, extending existing tests instead of adding new ones."
---

Entries restate the sites; the reviewer judges the committed range afresh.
