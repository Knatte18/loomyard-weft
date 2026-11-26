---
round: 1
focus:
  - "tools/machinecost/probe.go, newbinary.go, starts.go, gitload.go: the entry says the probe deletes every folder it makes and makes none outside the -dir folders and its own temp folder. Does that hold on every exit path, an interrupt or kill included, and for what the started go build, git and lym processes write on their own?"
  - "tools/machinecost/newbinary.go linkProbeProgram: do the build environment and its doc comment's claim that every machine links the program the same way hold, with GOFLAGS or other build settings set through the environment or `go env -w`? Do the timed first and second runs measure what the entry asks?"
  - "tools/machinecost/gitload.go: does the workload's environment keep a machine's or caller's git settings (inherited GIT_* variables included) out of the comparison? Does the per-step processes figure, counted from trace2 events, match the entry's number of processes started? Do the user and system figures cover what their names and comments say?"
  - "tools/machinecost main.go dirList.Set and report.go compare: how do the derived measurement names and the pairing behave for a bare path, a relative path, a path or name containing '=' or '/', and two reports whose configuration differs in more than the counts?"
  - "Scope against the entry: check each item of the amended board entry, Idea and Notes, against what the diff delivers, including the measurement reports under docs/benchmarks/ that _lym/landing/summary.md says are left out, with the record holding no Added after Discussion entry."
  - "Repository rules: does tools/machinecost follow PATTERN.md and pattern/ for a new tool (process spawning, git invocation, temp folders, test tiers and TestMain, file headers and doc comments), and do any docs they name for such a change need to land in this diff?"
  - "tools/machinecost tests: read the mutation survivors in report.go and main.go against the covering tests, check whether behaviors the summary claims (cleanup on failure, the different-counts warning, the traced round kept out of timings) are covered, and whether probe_test.go's leftover glob of the shared temp root can be affected by anything other than its own run."
---

Seed focus drawn from the board entry, the change description and a read of the diff.
The reviewed range is code only under `tools/machinecost/`, so the entries centre on measurement validity, cleanup, scope and test coverage.
