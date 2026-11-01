```yaml
round: 2
fixed: [F2]
disputed: []
deferred: []
```

## Changes

- **F2** (fixed as proposed), commit `972525a6`.
  In `internal/fabricengine/originreconcile_integration_test.go`, the subtest "A fetch that fails after both ls-remote calls answered is an error and moves nothing" takes `lockPaths := remoteRefsLockPaths(t, hub)` before the call.
  After the error, refs and HEAD assertions it fails when `freeRemoteRefsLocks` returns fewer paths than `lockPaths`, so the fetch-error return is pinned beside the three returns the "frees both remote-refs locks on every return" subtest covers.
  `lym gate test --tier 1 ./internal/fabricengine` passed.

## Out of the fix

- **F1** is `decision` class.
  It went to the parent, who recorded the repository-wide remote-refs lock as the settled way (creel #265).
  The fixer changed nothing for it.

## Disputed

None.

## Deferred

None.

## Self-check

All four forks returned a report; no fork returned none.

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.
  It noted that the error return of the second `HoldRemoteRefs` and the `judgeBranch` error return carry no lock assertion; the review did not ask for the first, and accepted the second as unpinned.
