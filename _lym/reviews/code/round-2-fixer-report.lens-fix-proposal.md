```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals for F1, F2 and F3 as the fixer carried them out.

**Proposals fixed as proposed:** F1, F2, F3. I built a counter-case for each.

**F1, `readHead` in the audit loop:** no finding.
- A landing between two audit jobs now sets `headMoved` and returns nil before the next build.
- That return leaves `r.queued` at 0, the same as the existing `r.headMoved` branch it sits in front of. The exit record stays consistent.

**F2, lock around `runproc.List` and `runproc.Register`:** no finding.
- The path `HubScratchDir/testmap-upkeep.lock` passes `refuseTrackedTree`, because the `.lym` segment resets `inBolt`.
- It is not under `RunProcDir`, so `runproc.List` and `runproc.Owners` never see it.
- `flock` blocks two callers in the same process as well as across processes, and `defer` releases it on every return.
- The blocking `AcquireWriteLock` can only wait for another caller's brief List and Register pair.

**F3, `slices.Contains` guard plus `addMeasure`:** no finding on correctness.
- An audit job on a package already built in the same call now adds its seconds and tests to that package's one entry, and the package stays in `built` once. That fits the report's "one entry per package" reading.
- The unconditional `r.built` append in `write` (the no-test and all-spawn paths) runs before the slot is taken. A yield cannot repeat it.
- The new row fails with the defect in place. `Built` would read `[yielding yielding]`, so `slices.Equal(pass.Built, []string{"yielding"})` fails.

**LOW: the new test row returns early.**
- Changed text: `if !handCall { ... return }` in `runPassYields`, `internal/gatecli/testmap_integration_test.go`.
- Summary: the `return` skips the tail assertion that `testcover.ReadExitRecord` holds the pass's PID and a queued count of 0. The resumed-pass row therefore never checks the exit record.
- Evidence: the `ReadExitRecord` check sits after the `afterPass` block, below the early `return`.
- Fix: put the exit-record check before the `if !handCall` block, or make only the `afterPass` comparison conditional.

Further tool calls: none. My context already held every changed file.
