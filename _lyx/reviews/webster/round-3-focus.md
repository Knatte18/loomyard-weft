---
round: 3
focus:
  - "internal/lyxtemp/sweep.go `Sweep` removal of a dead run directory while a second sweeper probes the same directory's lock through `ownerDead` (`lock.TryAcquireWriteLock`, which creates the lock file when absent): does a failed `os.RemoveAll` converge, is a path already gone treated as success, and is `TestSweep/two_concurrent_sweeps_both_succeed` left with its assertions intact?"
  - "The `ownerDead` and `Sweep` doc comments in internal/lyxtemp/sweep.go: do they state what the probe and the removal actually do under a concurrent sweeper?"
---

The focus covers the concurrent-sweep race that failed Publish's verify; the rest of the range is reviewed as usual.
