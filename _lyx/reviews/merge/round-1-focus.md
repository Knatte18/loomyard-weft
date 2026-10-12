---
round: 1
focus:
  - "internal/shedengine/run.go, goto.go, status.go: the task exported `MissingStatusClause` (was `missingStatusClause`) while the parent added `ParseStatusTolerant` beside it in status.go. Settle whether every caller on both sides, including the parent's reportengine and shedadapters code, still compiles to the intended name and whether the exported clause is used the way the parent's tolerant read expects."
  - "internal/shedverbs/loop.go: the task's `ReadStatus` seam and status-stop path against the parent's new status readers (`ParseStatusTolerant`, `shedrun.TreeRunRels`). Settle whether two readers of status.json now disagree on what counts as absent, undecodable or unreadable."
  - "PATTERN.md, pattern/PATTERN-cli-cobra.md and docs/overview.md, edited by both sides: settle whether the merged text keeps both sides' rules intact and whether the task's runnable-group audience rule contradicts the parent's new cli-cobra line, and whether the parent's new `internal/shortcutcli` and reportengine commands follow the audience rule the task wrote."
  - "pattern/PATTERN-leaf-packages.md, PATTERN-sole-parsers.md, PATTERN-told-geometry.md (parent-changed): settle whether the task's new code in planparser, gatecli and shedverbs follows them, notably any new parser of plan or verify text outside its sole parser."
  - "cmd/lyx/testdata/test-budget.yaml: settle whether the merged budget rows still match both sides' added top-level tests in internal/shedengine, internal/shedverbs and the packages the parent grew."
  - "Test economy across the merge: settle whether the parent's `ParseStatusTolerant` test and the task's status-read tests cover the same behavior twice, and whether the interaction of the loop's status-stop with the parent's changes has any uncovered behavior."
---

Seed focus drawn from the overlap note and a read of both sides' diffs in `internal/shedengine`, `internal/shedverbs` and the overlapping docs.
