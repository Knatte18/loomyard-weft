```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, the test that pins it, or none.

**Per behaviour**

1. `deployStale` runs the command when Command is set and Behind is true, before the owner write: pinned by `TestAdopt/RootDeploysALymOlderThanThePrimesHeadBeforeTheOwnerWrite` (`adopt_test.go`), which checks the command, the order and the `Deploy` outcome.
2. `deployStale` skips when Behind is false: pinned by `ALymBuiltFromThePrimesHeadIsNotDeployed`.
3. `deployStale` skips when Command is empty: pinned by `NoDeployCommandForThePrimesBranchDeploysNothing`.
4. A failing deploy refuses with the holds standing and no owner write, beam steps or check, and the `Deploy` outcome with its log survives on the error return: pinned by `AFailedDeployRefusesBeforeTheOwnerWriteWithTheHoldsStanding`. The row does not assert `Partial` or the mutation record on that return.
5. An unreadable deploy state becomes a warning and the adopt goes on: pinned by `AnUnreadableDeployStateIsAWarningAndTheAdoptGoesOn`.
6. `Deploy` appears in the envelope only when set: pinned by `TestAdoptFields/ARunDeployIsTheDeployField`, which asserts both presence and absence.
7. A beam adopt under a prime this machine already owns deploys nothing: pinned by `ABeamAdoptUnderAPrimeOwnedHereDeploysNothing`.
8. `installedBehind`: pinned by `TestInstalledBehind`, with rows for ancestor, modified ancestor, head itself, commit absent, not an ancestor, no stamp and ancestry error.
9. The `rootGoing` gate when root is named but a stop holds root back (`held[root]`): none. No row sets a root-held stop together with a stale build, so the gate could drop `!held[root]` and no test would fail. The deploy would then run although root is not going.
10. The `rootGoing` gate when root is named and the prime is already owned here: none. This reaches `deployStale` through `rootNamed`, and the intended behaviour is not pinned either way.
11. `readDeployState`: none. Nothing covers the empty `deploy` short-circuit, a branch missing from `deploy_branches`, the `OfFile` error, or passing `fab.CommitExists` and `fab.IsAncestor` through. Only the faked `DeployState` is exercised.
12. `hubDepsAs` wiring of `DeployState` and `RunDeploy`: none. `export_test.go` builds the real deps, but no test drives a root adopt through them.
13. `runAdoptDeploy`: none. This covers the `testing.Testing()` refusal, the scratch directory and log path, and the mapping of start error, timeout and non-zero exit to messages. The `testing.Testing()` guard makes it unreachable from any test. `TestRunDeploy_WritesTheLogAndReportsExitCodeAndTimeout` covers only `RunDeploy`'s own behaviour.
14. `deployStale` with nil `DeployState`: none. This is a trivial branch.
15. `fabricengine.Fabric.CommitExists`: none. It is a one-line delegation, and `SHAExists` is presumably tested in `gitrepo`.
16. Renamed `RunDeploy` and `DeployTimeout`: the existing tests were updated (`deploy_integration_test.go`, `land_test.go`), so they still pin the behaviour.
17. Help text and doc changes: static prose, exempt.

**Findings**

- **MEDIUM, design.** Location: `internal/adoptcli/adopt.go:333` (gate), `adopt_test.go` rows around line 904. Summary: the `rootGoing` gate has no row for root named while a stop holds root back, so a deploy that fires without an owner write is not pinned. Evidence: no TestAdopt row combines a held root with `w.deploy = staleBuild`.
- **MEDIUM, design.** Location: `internal/adoptcli/deploy.go:67-103` (`readDeployState`). Summary: the real state read has no test. This includes the branch-list and empty-command short-circuits and the Behind judgement wired through fabric. Evidence: `deploy_test.go` covers only `installedBehind`, and the rows in `adopt_test.go` fake `DeployState`.
- **LOW, design.** Location: `internal/adoptcli/deploy.go:121-140` (`runAdoptDeploy`). Summary: the error mapping and log path are untested, and the `testing.Testing()` guard prevents testing them. A seam such as an injected run function would make them testable. Evidence: the guard at line 122, and no test names `runAdoptDeploy`.
- **LOW, design.** Location: `internal/adoptcli/adopt_test.go`, the failed-deploy row. Summary: the row does not assert `Partial` or the mutations on the refusal return, which the focus asks about for `finish()`. Evidence: the row checks only the owner request, the steps, the holds and `Deploy`.
- **NIT, design.** Location: `internal/fabricengine/warpforward.go:41`. Summary: `CommitExists` has no test of its own, a thin delegation.

I made no tool calls.
