---
round: 1
focus:
  - "internal/proc/memorycap_linux.go and memorycap.go: does the `systemd-run --scope` wrapper still let the timeout's process-group kill reach the run, and does the `prlimit` RLIMIT_AS fallback apply only when the cached probe fails, at parallel 1?"
  - "internal/proc/memorycap_windows.go, namedjob_windows.go, tied_windows.go: is JOB_OBJECT_LIMIT_JOB_MEMORY set on the job from createKillOnCloseJob, and does a cap kill stay distinguishable from a timeout kill?"
  - "internal/testcover/runner.go: does Toolchain.command apply MemoryLimit to every test-binary run, and does RunResult.MemoryCapped get set only on a real cap kill, with a capped unmutated run setting FailsUnmutated?"
  - "internal/gatecli/redundantmutants.go: is a capped mutant counted as a kill and never rerun alone, is the mutant ID logged at spawn, does the answer list capped and timed-out mutants (file, line, operator, test), and is the timeout floor lowered from 10 s to 2 s?"
  - "internal/gatecli/redundantmutants.go clamp and internal/proc/memorylimit.go: is parallel clamped to memory headroom divided by the cap, reading memory.high and memory.max up the cgroup ancestry, with sane behaviour when no limit is found or headroom is below one cap?"
  - "internal/gateslot/config.go, template.yaml, doc.go: does redundancy_test_memory_mb default to 2048 and sit beside redundancy_test_timeout_sec, with validation of zero or negative values?"
  - "cmd/testtiming/redundancy.go and main.go: does the redundancy report step now carry the same memory and time bounds, and do gate testmap callers get the cap through RunAlone?"
  - "Tests (internal/proc, testcover TestRunAlone, TestGateRedundant, TestClampParallelToMemory, TestLoadConfig): does every allocating fixture stop itself at a few hundred MB, so a broken cap fails the test and never the machine? Check the redundancy report's contained candidates and test-economy bullets."
  - "Scope: does any test or code run lym gate redundant or lym gate testmap on a real package of this repository, against the record's Bounds section? Do docs/overview.md and the package doc.go files match the behaviour the diff lands?"
---

Focus follows the record's Fix and Tests sections, ordered by where a wrong cap would hurt most: the process-kill paths first, then the result plumbing, then config and tests.
