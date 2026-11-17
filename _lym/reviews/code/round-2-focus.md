---
round: 2
focus:
  - "internal/proc/memorycap_linux.go, runMemoryCapped scope path: when systemd-run cannot create the scope (for example the run's cmd.Env lacks the user bus), does RunMemoryCapped return a non-ExitError error rather than the command's exit, as its doc in internal/proc/memorycap.go says a cap that could not be set is reported? Does a test cover that case?"
  - "internal/proc/memorycap_linux.go, probeMemoryScope: is the systemd-run probe bounded in time, and does a timeout fall back to the data segment limit?"
  - "internal/testcover/runner_test.go TestRunAlone and internal/testcover/testmain_test.go runStandIn: does a row run the outOfMemoryLine branch of RunAlone with MemoryLimit set on a host whose scope probe passes, and want MemoryCapped true?"
  - "internal/gatecli/redundancy.go report step (the result.MemoryCapped case with reasonMemoryCapped) and internal/gatecli/testmappackage.go (the MemoryLimit field on testmap's runner): does any existing test fail if the case or either MemoryLimit field is dropped?"
---

Focus covers where the fixes to the gating design findings land.
