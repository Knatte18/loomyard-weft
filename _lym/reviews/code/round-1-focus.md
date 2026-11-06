---
round: 1
focus:
  - "internal/driftengine/verify.go and judgesnapshot.go: does `resumeJudgeSession` order the snapshot and the `scope.SnapshotLost` write so lym's own `scope.json` and `judged.json` writes never read as the judge's, while a judge edit to a task file or a record it may not touch still halts?"
  - "judgesnapshot.go and the halt texts in verify.go: does any way-forward text still tell the reader to delete the snapshot directory, and does a resume with the snapshot gone take a fresh one only when no live judge session was guarded, keeping the halt otherwise with that said?"
  - "judgesnapshot.go and verify.go lifecycle: can the snapshot's removal or creation race a resume?"
  - "diff.go, diffmode.go, config.go, template.yaml: does the hub-wide volatile files and keys list in `drift.yaml` load, validate and apply in every diff mode, hard-code no harness file name, and get documented in the template? Does the change description say what the Models hub must set?"
  - "config.go, hubreconcile, preflightshed/preflight.go, shedrecipe: does reconcile remove the whole `-e NAME={skip_file}` argument from both capture and compare commands, does preflight or config load find a leftover before Drift-Recheck, and does the error's way forward work from a beam orch that is refused hub-wide config writes?"
  - "Test economy across the new tests in verify_test.go, config_test.go, diff_test.go, diffmode_test.go and judgesnapshot_test.go: duplicated coverage, and behaviour the diff adds with no coverage."
---

Seed focus from the Goal's four bugs and the diff's file spread.
