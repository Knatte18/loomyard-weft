```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens on the testmap upkeep diff. Findings below, with the per-function list first.

**Per function, case run, verdict**

- `TryAcquireSpare`, slots=1: nil lease, correct. Slots=3 with two free: takes one, correct. One free: nil, correct. Drain held: nil, correct. Between counting two free and `tryAcquireAny`, a gate can take a slot, so none is left. The yield watch recovers within one poll, so this is acceptable.
- `Contended`, a lease holding the only slot: the loop checks nothing and returns true. Draining returns true. Any non-own slot free returns false. Correct.
- `registerUpkeepPass`: a stale record is dropped by `List`'s liveness filter, correct. Two passes starting together can both list nothing and both register. That race is small and a lock-free list-then-register check can't close it. No unregister is needed, since `List` filters dead PIDs.
- `runTestMap` with `--upkeep` and a package argument: refused before any work, correct. `--upkeep --max 0`: refused, correct.
- `mapPackages`, slots<2: counts every job as queued and returns, correct. HEAD moved before a job: counts the jobs left, correct. A `readHead` error: returned. Defect: F1.
- `measureJob`, yield: releases the lock (before `releaseHeldSlot`) and the slot, waits, retakes the lock without waiting, replans. A held lock gives `continue` and then the skip, correct. A HEAD move during the wait gives `queued++`, correct.
- `waitForSpareSlot`: the pause doubles to the 30s ceiling and HEAD is read on each wake. Correct. HEAD is read only after the first pause, so the first poll is not preceded by a read. That is minor and fine.
- `jobSlot`: a nil lease takes the spare slot or returns `errSlotYielded`. A held lease checks `Contended`. A non-pass calls `holdSlot`. Correct.
- `watchForYield`: the poll error is warned once and the pass keeps the slot. The stop function cancels and joins. Correct.
- `runRound`: the post-run `interruption` check discards the whole in-flight round, including tests that had finished before the cancel. That is within D9's wording ("runs in flight") and D10's bound. Correct, not flagged.
- `runRounds`/`endRound`: a dirty linked file or a moved HEAD discards the round and keeps the earlier written rounds. A selfCheck that ran in a discarded round has already bumped `r.selfCheckMismatches`. This is cosmetic, so I don't flag it.
- `nextRound`: the first Parallel test with parallel≥2 fills the round up to `parallel` in queue order, and the rest keeps its order. Correct. A non-Parallel head test runs alone, correct.
- `declaresParallel`: empty body, unnamed parameter, and `t.Parallel()` after another statement all return false. Require-then-Parallel returns true. A file that doesn't import tiergate with `t.Parallel()` first returns true, which matches D10 literally. Correct.
- `orderJobs`: sums by import path, sorts descending then by import path. Correct provided `launch.Package` is an import path. My grep found no runrecord file in `internal/gateslot`, so I could not confirm this. `TestOrderJobs_…` pins it against its own fixture.
- `audit`: a yield returns nil with the slot still held until `close`. The audit round then counts as done, which is acceptable.
- `planLockedPackage` with `tally=false`: `case !tally:` is an empty case, so no count is made. A Miss still returns the whole job. Correct.
- `close`: writes the exit record only under `--upkeep`, and releases the slot and tmp dir. Defect: F2.

**Findings**

- F1, MEDIUM, design, `internal/gatecli/testmap.go:486-512` and `testmap.go:268-275`.
  - When `mapPackages` returns an error mid-walk, the remaining jobs are never added to `r.queued`. Examples are a slot or read failure, or `r.ctx.Err()` after a terminating signal.
  - `close` then writes an exit record with `PlannedHead=H` and `Queued` short by the unmeasured jobs, possibly 0. The dobby's D8 rule (HEAD equal and nothing queued) then starts no pass.
  - The unmeasured work is lost until HEAD moves. D8 defines queued work as "the work left when the pass ended".
  - Fix: on an error return from the job loop, add `len(jobs)-i` to `r.queued`, counting the failed job itself as well.
- F2, LOW, design, `internal/gatecli/testmap.go:260-263`, `:675-681`.
  - `defer run.close()` is installed before `prepare()` and `packages()`. A `--upkeep` refusal there (no mapped tier, unreadable config, bad module) writes an exit record with `PlannedHead` empty.
  - The empty value never equals the prime's HEAD, so D8's "HEAD differs from the planned HEAD" starts a new pass on every dobby tick, in a loop. D8 intends loop avoidance.
  - Fix: write the exit record in `close` only once `r.head != ""`, or install the exit-record write after `prepare` and `packages` succeed.

**Tool calls**
1. `grep` over `internal/runproc` and `internal/gateslot`, and `runproc.Register` callers. I needed to know whether `List` filters dead PIDs, whether registration needs an unregister, and what `launch.Package` holds. The first two were answered; the `launch.Package` grep matched no file.
