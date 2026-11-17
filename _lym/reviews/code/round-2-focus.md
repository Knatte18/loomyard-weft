---
round: 2
focus:
  - "internal/proc/memorycap_linux.go `runMemoryCapped` and internal/gatecli/tierrun.go `runInvocations`: settle whether a tier whose shell is SIGKILLed by the process cap, or by any outside sender, can still print the memory_mb cap line, and whether the kill is attributed from the scope's kernel OOM-kill count."
  - "internal/gatecli/memoryguard.go, internal/testcover/runner.go `RunAlone`, internal/gatecli/redundantmutants.go and internal/gatecli/redundancy.go: settle whether a test binary the memory guard killed can be taken for a kill at redundancy_test_memory_mb, counted as a mutant kill, or set FailsUnmutated."
  - "internal/gatecli/tierrun.go `runInvocations` and internal/gatecli/child.go: settle what a tier run reports, and whether the report names a way forward, when the memory_mb scope never runs the command or the capped start fails before the shell starts."
  - "internal/gatecli/tierrun_test.go and the _linux tier-run tests: settle whether a memory guard kill, and a kill at memory_mb, are pinned through exit code, log, causes, the memoryStopped flag and the no-rerun rule."
  - "internal/gatecli/run.go `runTest`: settle whether the redundancy report step's test binaries run under the memory guard, as the board entry's guard over every test binary lym starts asks."
  - "internal/gatecli/tierrun.go and internal/gatecli/memoryguard.go: settle whether a guard kill in a tier run without -run names the test that was killed."
  - "internal/gatecli/doc.go Memory guard and CPU watchdog sections, and internal/gatecli/cli.go stderr field comment: settle whether each sentence about what the kernel cap or the guard watches, and each list of lines printed to stderr, matches the code."
---

Round 2 reviews the tree as the fixer leaves it; each entry names where to look and what to settle.
