---
round: 1
focus:
  - "internal/gatecli/memoryguard.go `kills`: does the sum of the sampled tree's RSS compared with `limit.Used` tell the run's own growth from other load? Read what `proc.ReadMemoryLimit` counts in `Used` and what `proc.TreeSample` puts in the tree (reparented processes, shared pages, tmpfs or page cache the run produced)."
  - "Every caller of the memory guard (tier runs, `--copies`, report steps, `redundant`, `mutate`, `--map`): which root and `foreign` set does each give it, and can memory this same call or its concurrent runs hold be counted or named as other load?"
  - "internal/gatecli/tierrun.go `slicePressure` and `stoppedOnSlicePressure`: can a run that also failed for a reason the code under test caused, but with an empty `record.Failures` (package-level failure, panic, TestMain exit, build or vet error in the same command), be classified as judging no code, and so be rerun, skip `--stop-on-fail` and spawn no fixer?"
  - "internal/gatecli/run.go, copies.go and tierrun.go `retryTier`: trace mixed outcomes (pressure stop with shortage, crash, timeout, process cap, memory cap or cancel, across run, rerun and copies). Do the flags, the failed-run labels, the closing lines and the records agree with each other and with the package doc?"
  - "Readers of a gate call's verdict outside `detachfixer.go` (callers of `gateslot.FindRunnerCrash`, the loom verify step, landing gates, `lym webster verify`): measured against the board entry's asks (wait and retry or report not judged, with a way forward that fits), what does each do with a memory pressure stop?"
  - "internal/proc/proc_linux.go `ResidentInCgroup` and `otherHolders`: check the cgroup path form matches `MemoryLimit.Cgroup`, and whether the named holders can be misleading (kernel or exited processes, the verb itself or its ancestors, unreadable entries)."
  - "The board entry asks to check the memory use of `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain`: does what the diff and `_lym/landing/summary.md` offer satisfy that ask, and do the summary's other claims, including `## Merge danger`, match the diff?"
---

Seed focus for round 1, drawn from the board entry, the review range's diff and commit messages, and the change description.
No finding has been judged; each entry is a question to settle.
