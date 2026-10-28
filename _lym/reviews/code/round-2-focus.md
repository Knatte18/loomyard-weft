---
round: 2
focus:
  - "internal/shedverbs/loop.go runChild and internal/shedverbs/loopchild.go startChild: settle whether every failed child start, not only an ETXTBSY one, records a cause reading `start-failed after <n> tries: <error>` with n the attempts actually made, including a backoff cut short by a failed `LoopSpec.Sleep`."
  - "internal/shedverbs/loop.go runLoop trace fallback: settle whether an interrupted stop whose child left no trace file and no non-empty stderr, whether the child never started or started and died, records the loop's own trace file, and stays empty without a panic when `LoopTraceFile` is nil or returns an empty string."
  - "internal/shedverbs/loop_test.go TestLoop_ChildStartRetriesABusyExecutable and tools/deploy/main_test.go TestInstallBinary_RenamesTheBuildOverTheTarget: settle whether the cancelled-backoff, empty-path and rename-failure branches each have a test row, and whether those rows duplicate an existing test."
---

The first round's fixes change the cause text, the fallback condition and the `startChild` signature, so the doc.go wording and the test strings that pin them are worth checking against the new code.
