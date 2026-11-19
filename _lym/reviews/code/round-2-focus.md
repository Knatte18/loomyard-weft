---
round: 2
focus:
  - "internal/shedverbs/loopwait.go unmarkedExitCause and its callers deadLoopStop and loopExitedEarlyStop: does the cause of an unmarked loop exit rest on anything that moves with the wall clock or differs by platform, and is a cause stated as fact where the evidence only supports a likely one?"
  - "internal/proc and internal/shedverbs/looppid.go, the pid record the loop writes before its first step: does every reader of the record, including older files and pair teardown, tolerate the fields the diff adds or changes, and do the proc package doc and the shedverbs doc agree with the code?"
  - "internal/shedverbs/loopwait.go loopExitEvidence: which loop-log marks count as the loop's exit record, and does a loop that marked something other than an exit, such as a reconciliation of orphaned steps, and then died unmarked still get the unmarked-exit diagnosis?"
  - "internal/shedverbs/loop.go runLoop and endWithoutEnvelope: does every early return mark the loop log with a reason, and do the tests pin the returned exit code and each branch's reason text?"
---

Round 1 left open the unmarked-exit cause, the evidence classification, and test and doc gaps around the same code.
