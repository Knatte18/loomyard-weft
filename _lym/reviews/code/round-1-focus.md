---
round: 1
focus:
  - "internal/proc/memorycap_linux.go `scopeStartScript`: does the sh script run correctly under dash and bash? Settle the cgroup parse (`0::` line, `$scope` empty or unset), the `read max && [ ... ] || { ...; exit 125; }` precedence, quoting of `$1`, and whether `shift` before `exec \"$@\"` leaves the command's own arguments intact."
  - "internal/proc/memorycap_linux.go `runMemoryCapped`: does the holder always end? Settle every return path (start error, cancel, SIGKILL, wait error, ErrMemoryCap), that `release` closes on each, that `holderEnd` and `marker` close in the parent, and that a holder left in the scope cannot keep `cmd.Wait` or the scope's `cgroup.kill` cleanup from finishing."
  - "internal/proc/memorycap_linux.go: does the `wantMax` check (limit rounded down to a page) match what the kernel writes to `memory.max` for every limit the callers pass, including one not a page multiple? Settle whether a legitimate cap can now be refused with ErrMemoryCap."
  - "internal/proc/memorycap_linux.go: the decision record asks that a capped command never runs uncapped silently. Settle whether any path still runs the command without the check (the non-scope or fallback path, the `RunMemoryCappedVia` entry, a user manager that ignores MemoryMax) and whether the error text names the cause."
  - "_lym/landing/summary.md claims against the diff: the root cause (the unreported kill read raced the scope's removal, not a failed cap), the claim that the machine hang was a probe pipeline stage outside the scope, and the 50-run pass beside other gate work. Check each claim the diff can bear out, and that the change description names root cause and fix as the decision record's Done-when requires."
  - "internal/proc/memorycap_linux_test.go and export_linux_test.go: test economy and bounded hogs. Settle that every hog is fixed-size and time-bounded, that the new row in the tree-kill test and the renamed scope-start test cover new behavior without duplicating each other, and whether the `TestRunMemoryCappedVia_StartsThroughItsStartAndReportsAChildKilledAtTheCap` redundancy entry (not judged, out of process) needs a `//testtiming:keep` mark."
  - "internal/proc/doc.go and the code comments in memorycap_linux.go: do the doc statements (check before run, read after the command ends, holder keeps the scope) match the code, and do removed items (`watchForOOMKill`, `poll`, unused imports or constants such as `errors`, `cgroupMemoryEventsFile` users) leave no dangling reference?"
---

Seed focus for round 1 over `0b42877c..HEAD`.
The answer key is the decision record alone; its Goal is the board entry, with no added entries.
