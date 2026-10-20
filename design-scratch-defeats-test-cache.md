# Scratch files under .scratch change the worktree root listing and defeat go test cache hits

`scratch-defeats-test-cache` · Note · bug, gate

Imported from [issue #771](https://github.com/Knatte18/loomyard/issues/771).

**What happened**
While measuring `lyx gate test` cache hits inside a task worktree, each new output file created under `.scratch/` changed the worktree root's directory listing.
The source-scan tests read that listing and Go keys its test cache on it, so those packages missed the cache and needed an extra pair of module-wide runs.
Writing into output files created beforehand avoided the misses.

**Why it is abnormal**
The agent prompt directs scratch files to `.scratch/` in the working directory, yet creating files there invalidates the cache the gate depends on, which skews measurements and wastes gate runs.

**What to do**
Keep `.scratch/` out of what the scans read and Go keys on, or document that scratch files must be created before the first gate run.

Provenance: task `gate-fewer-new-binaries`, note `Darn-Write.md`.
