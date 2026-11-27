# Memory guard fails a gate by killing cmd/lym tier1 when the user slice is full from elsewhere

`memory-guard-infra-stop` · Note · bug, gate

Imported from [issue #267](https://github.com/Knatte18/loomyard/issues/267).

## What happened

The memory guard killed the `cmd/lym` tier1 test binary in repeated gate runs: two `lym gate test --gate darn-write` runs by a gate fixer, and one run of the loom verify step.
In the gate fixer's runs, `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain` was running each time.
The branch's changed test files were all outside what the `cmd/lym` test binary builds.

The guard's output (paths made repo-relative):

```
lym gate: memory guard: killed the test binary lym.test of the package in cmd/lym, with every process it started: the memory limit of /user.slice/user-1000.slice had 1635 MB left of 16384 MB, and it was the largest test binary at 2027 MB
lym gate: memory guard: killed the test binary lym.test of the package in cmd/lym, with every process it started: the memory limit of /user.slice/user-1000.slice had 1199 MB left of 16384 MB, and it was the largest test binary at 1606 MB
lym gate: memory guard: killed the test binary lym.test of the package in cmd/lym, with every process it started: the memory limit of /user.slice/user-1000.slice had 1516 MB left of 16384 MB, and it was the largest test binary at 1199 MB
```

Each message ends:

```
its tests fail; way forward: find the test that holds memory without bound, fix it and re-run the same command; raise gate.memory_quit_mb through "lym config gate" from the prime only for a package whose tests need that much
```

After the red tier, all the follow-on steps judged nothing:

```
gate test: reach report: a tier of the call was red, so the step judged nothing
gate test: redundancy report: a tier of the call was red, so the step judged nothing
gate test: mutation report: a tier of the call was red, so the step judged nothing
```

The final verify log of the run has no memory-guard kill.

## Why it is abnormal

- The killed binary held 1.2 to 2 GB of a 16 GB slice, so most of the slice's use lay elsewhere. The cause of that use was not measured.
- The run reports the kill as a test failure, although no test failed, and a change that cannot affect `cmd/lym` gets a red gate.
- The way forward asks for a fix to a test that holds memory without bound, or a higher limit set from the prime. A gate fixer in a task worktree can do neither when the pressure comes from other processes, so it left the gate red.

## What to do

- Have the guard tell slice-wide pressure apart from one binary's growth, and wait and retry, or report the run as not judged, instead of failing the tests.
- Name the other large holders of memory in the kill message, and give a way forward that fits that case.
- Check the memory use of `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain`, which was running at the time of the gate fixer's kills.

Provenance: task `prebootstrap-board-proc-leak`; note `gate-fixer-darn-write.md`.
