---
round: 1
focus:
  - "internal/gatecli/tierrun.go, run.go: does the crash detection match the board entry (no test result plus a signal exit such as 134 or 139), or does it misclassify a tier with failing tests, a timeout kill or a resource-shortage exit as a crash, or miss one? Check the exit-code arithmetic (128 plus signal) and the no-failing-test condition."
  - "internal/gatecli/run.go, tierrun.go: the one rerun. Is it capped at one, does it write both runs' records, and does it interact correctly with the existing resource-shortage rerun so a tier cannot loop or rerun twice?"
  - "internal/gateslot/runlog.go and internal/verifytree/verify.go: the RunnerCrashLine written and the line read back into Result.RunnerCrash. Does the writer and reader agree exactly, and does a call with one crashed and one genuinely failed run get judged a code failure?"
  - "internal/landingshed (finalize.go, fixerrun.go, parentalign.go, verifygate.go) and internal/loomshed/verifygate.go: on a repeat crash does every path stop with the crash named and spawn no fixer, and does a real failing-test result still reach the fixer? Look for a path that still spends a fixer attempt or writes a landing failure record on a crash."
  - "internal/gatecli/detachfixer.go: the card fixer's handling of a crash verdict. Is a crash never handed to a fixer, and is the stop reported with the crash named?"
  - "Test economy: do the new and changed tests in tierrun_test.go, watchdogline_test.go, finalize_verify_test.go and verifygate_test.go each cover a distinct behavior, and is any added behavior (rerun record shape, loom row terminal failure, merge fixer turn gate) left uncovered?"
  - "PATTERN.md, doc.go files and the commit messages: does the PATTERN.md change and each doc.go edit match the diff's behavior, and do the commit-message claims hold against the code?"
---

Seed focus for the first round: the diff implements the board entry's three expectations (crash reported apart from failure, one rerun with both runs recorded, a repeat crash halting with the crash named rather than a fixer).
The decision record carries only `## Goal`, so the amended board entry is the measuring stick.
