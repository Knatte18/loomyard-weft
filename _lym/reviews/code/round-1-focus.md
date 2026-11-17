---
round: 1
focus:
  - "internal/gatecli/memoryguard.go: does the guard SIGKILL the whole test binary's tree on RSS over the cap or on the user slice nearing memory.high, and does its report name the killed test and package? Check the failure paths (unreadable /proc or cgroup files, a process exiting mid-sample, a stopped sampler goroutine)."
  - "internal/gatecli/tierrun.go, child.go, redundant.go, testmap.go: does every test binary lym starts (gate tiers, RunAlone runs of redundant and testmap) go through the guard, and do the gate tiers' go test runs go under proc.RunMemoryCapped with a per-tier setting? Look for a path that bypasses either."
  - "internal/gateslot/config.go, tiers.go, template.yaml, doc.go: is the per-tier memory setting parsed, defaulted and validated consistently, and does a capped tier run report its kill distinctly from a test failure?"
  - "internal/proc (memorycap*.go, proc_linux.go, proc_windows.go, sample.go, cpu.go): the diff moves and deletes helpers across platforms. Does the Linux, Windows and non-Linux build each stay coherent, with no dangling reference to cpu.go's removed code and no behavior change to the existing RunMemoryCapped contract?"
  - "internal/gatecli/watchdog.go and pattern/PATTERN-daemon-wakeups.md with cmd/lym/daemonwakeups_test.go: does the new sampling wakeup land as the PATTERN entry declares, and does the watchdog's CPU trigger behave as before?"
  - "docs/overview.md and package doc.go files: do the docs the change touches state the new behavior accurately against the code, and do the commit messages and any summary claims match the diff?"
  - "Tests (memoryguard_test.go, config_test.go, memorycap_linux_test.go, proc_linux_test.go): does each new behavior have coverage, and does any test duplicate another? The redundancy report lists every candidate as unique or not judged, so file no redundancy finding on that ground."
---

Seed pass: the decision record holds only the board entry under `## Goal`, so the entry's Idea section is the measuring stick.
