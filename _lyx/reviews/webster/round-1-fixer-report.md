# Webster round 1 — fixer report

All ten findings are fixed, one commit each, not pushed.

| Finding | Commit | Change |
|---|---|---|
| F1 (MEDIUM) | `fb5f8640a` | `TestAddRollback_UnwiresJunctionsOnPostWiringFailure`, `TestAdd_GitFailureCarriesGitsOwnReason` and `TestAddRollback_CreatedPathLeavesNoOriginRecord` now pass `AddOptions{SkipPush: true}` and assert the error names the step-11 `push branch` failure, so a pre-flight refusal can no longer make them pass vacuously. |
| F2 (LOW) | `8e545b0f2` | `probeWarpLeftover` takes `slug` and sets `ErrRemoteLeftover.Slug`; `TestAdd_WarpLeftoverRefusedAtPreflight` asserts it. |
| F3 (LOW) | `02862fc08` | The lease-race hook records origin's tip after its push; the test asserts the hook ran and that origin still holds that tip after Add fails. |
| F4 (LOW) | `e369b9595` | Rewrapped the `leaseSHA`, `warpBranch`, `archiveTag`, `ownedPairWeftBranch`, `dirtyArchivedOnRemote` and `resolvePairWeftBranch` comments in `destroy.go` with semantic line breaks. |
| F5 (LOW) | `cab54a02d` | Rewrapped the `remoteleftover.go` file header. |
| F6 (LOW) | `4f5635f98` | Rewrapped the `DeleteRemoteBranchLeased` doc comment. |
| F7 (LOW) | `359b5e7a4` | Rewrapped the new comments in `export_test.go`, `destroy_test.go`, `destroyremote_integration_test.go`, `deleteremotebranch_integration_test.go` and `addleftover_integration_test.go`. |
| F8 (NIT) | `80acca9df` | `add.go`: the header sentence now sits on one line, and the probe comment breaks before ", so". |
| F9 (NIT) | `85f11061b` | Each of the three tests the diff switched to `SkipPush` gets a comment explaining that `SkipPush` skips the pre-flight probes while the step-11 warp push, which ignores it, still fails. |
| F10 (NIT) | `df3091ecd` | The `destroyremote_integration_test.go` header now says "every case" instead of "either case". |

Verification after the last fix: `go build ./...`, `go vet -tags integration` over the three packages and `go test ./...` all pass.
`go test -count=1 -tags integration` passes for `internal/gitrepo` and `internal/fabricengine`.
`internal/fabriccli` fails only `TestRunCLI_CleanupRemoteFailureExitsNonZero`, which fails the same way on an export of the merge base `676d9ec03`, so this diff did not cause it.

## Deferred

None.
