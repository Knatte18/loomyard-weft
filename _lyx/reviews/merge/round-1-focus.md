---
round: 1
focus:
  - "internal/pauserecord/record.go and hold.go (new in the parent's pause/resume commit) read and rewrite JSON records with json.Unmarshal, json.MarshalIndent and a hand-rolled atomic write, outside internal/state: settle whether PATTERN-tolerant-records (a rewrite goes through internal/state, or the type keeps unknown keys itself) applies to them, and whether an older binary rewriting a pause record would drop a newer binary's keys."
  - "Parent-added code that reads or writes shedengine.Status or other run records (internal/shedverbs/loop.go, internal/shedengine/goto.go, rename.go, run.go, internal/battencli/wire.go, arm.go, internal/loomcli/resume.go, start.go, looparm.go): settle whether each uses the task's renamed state read functions (ReadJSONNoMkdir) and tolerant decode, and whether any new strict decode or direct os.WriteFile of a run record bypasses state's unknown-key keeping."
  - "internal/alignment/session.go and session_test.go: the task routes attempt.json through internal/state, and the parent also changed this file; settle whether the merged session code still writes through state and whether the parent's pause handling reads or writes attempt.json another way."
  - "internal/websterengine/runlevel.go, beginbatch.go, doc.go and verifygate_test.go: the parent removed webster's own pause code while the task changed the verify gate (no rerun after a hang) and the state tests; settle whether the verify gate, retired-key handling and package doc still agree after both edits."
  - "PATTERN.md, pattern/PATTERN-cli-cobra.md, PATTERN-daemon-wakeups.md and PATTERN-leaf-packages.md changed in the parent, and cmd/lyx/daemonwakeups_test.go plus testdata/test-budget.yaml were edited by both sides: settle whether the task's code follows the parent's new pattern rules, and whether the merged test-budget rows match the merged test counts and no test is made redundant by the pause/resume tests."
---

Seeded from the merge's overlap: the parent's pause/resume work added new run-record readers and writers, the task made every Go-written JSON record tolerant and key-keeping.
The decision record names the scan in `cmd/lyx/tolerantrecords_test.go`, which only catches `DisallowUnknownFields`, so a rewrite that bypasses `internal/state` is a gap the scan does not cover.
