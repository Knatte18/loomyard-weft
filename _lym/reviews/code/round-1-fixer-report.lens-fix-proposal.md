```yaml
lens: fix-proposal
outcome: reported
```

no findings

- F1, the non-nil `TaskLimit` assertion in `TestHubDobbyDeps_PrimeTidyShuttle`:
  - Dropping `TaskLimit: proc.ReadTaskLimit` leaves the func-valued field nil, so the check fails. The defect is caught.
  - The test never calls the seam, so it cannot read the host's real cgroup. Reading it would make the test host-dependent.
  - The extra prime-side assertion is correct. The tier1 run passed, so both geometries fill the seam.
- F2, the per-start read subtest in `TestStartReady_TaskLimitHold`:
  - A once-per-tick read sees only the free reading, so both tasks spawn. The test expects one spawn and one hold line, so it fails with that defect in place.
  - The `reads` counter is touched only from the tick, on the test goroutine, so there is no race.
  - The harness default `maxRuns` is 3, so `a` and `b` both reach the limit check. The `break` after the hold gives exactly one hold line.
  - The expected log order, `start a` and then the hold line, matches the loop order.
  - A counter that counts reads per call, not per start, would also pass if the daemon read the limit for non-ready tasks. That is outside the finding's premise, so it is not a defect of this fix.

Further tool calls: none.
