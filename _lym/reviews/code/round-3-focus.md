---
round: 3
focus:
  - "internal/proc/memorycap_linux.go, runMemoryCapped and the goroutine it starts to watch the scope's memory.events: settle whether any periodic wake in that code runs faster than once a second for the life of a capped command, and whether the way a kill at the cap is detected can still miss a kill just before the scope is removed."
  - "internal/proc/memorycap_linux.go, scopeCgroup and markerReadBound: settle whether the scope's cgroup listing is read while systemd-run may still be starting the scope, and whether a slow start can make runMemoryCapped return ErrMemoryCap for a command that ran."
  - "internal/proc/memorycap_linux.go, scopeHadOOMKill: settle whether a kill is reported as a kill at the tier's memory_mb only when the scope's own limit was hit, not an enclosing slice's limit or a host-wide OOM."
  - "internal/gatecli/memoryguard.go, startMemoryGuard and its kill-noting path, read with RunAlone in internal/testcover/runner.go: settle whether GuardKilled is true for every Go test binary in the chain above a binary the guard killed, so an outer binary that fails only because its inner binary died is never judged as a mutant kill or an ordinary failure."
  - "_lym/landing/summary.md against internal/gatecli/run.go, internal/gatecli/tierrun_memorycap_linux_test.go and the guard's test-naming code: settle whether each claim about the redundancy report step's guard, how the killed test is named, and what tests drive a real memory_mb runaway matches the diff."
  - "internal/gatecli/doc.go, docs/overview.md and internal/gateslot/template.yaml, the memory_mb and guard-kill clauses: settle whether they hold for lym gate testmap's recorded outcome of a guard-killed test and for a host whose user manager cannot cap a scope."
---

Round 3 re-reads the areas the round-2 findings touched, and nothing else.
