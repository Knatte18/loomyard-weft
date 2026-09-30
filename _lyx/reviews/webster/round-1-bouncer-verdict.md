---
verdict: APPROVED
rationale: "All ten round-1 findings (F1-F10) are fixed by commits fb5f8640a..85f11061b and verified in the diff; build, vet and integration tests pass for gitrepo and fabricengine, and the one fabriccli failure (TestRunCLI_CleanupRemoteFailureExitsNonZero) is in Cleanup's archive path, which this diff does not touch."
---

Review range: `git diff $(git merge-base main HEAD)..HEAD`, from `product.parent: main` in `_lyx/shed/fabric-readd-weft-push/status.json`.
It covers Cards 1–5, the boundary pin `fd2c3d3ec`, and ten fix commits, one per round-1 finding.

The round-1 report has no `## Focus departures` section, so there is nothing to ratify or reject.

## Findings, checked against the fixed diff

- **F1 (MEDIUM), fixed.**
  `TestAddRollback_UnwiresJunctionsOnPostWiringFailure`, `TestAdd_GitFailureCarriesGitsOwnReason` and `TestAddRollback_CreatedPathLeavesNoOriginRecord` now pass `SkipPush: true` and assert the error contains `push branch`, the step-11 text at `add.go:252`.
  They can no longer pass on a pre-flight failure.
- **F2 (LOW), fixed.**
  `probeWarpLeftover` takes `slug` and sets it on its refusal, and `TestAdd_WarpLeftoverRefusedAtPreflight` asserts `Slug`.
- **F3 (LOW), fixed.**
  The lease-race hook records the moved tip after `pushCommitToOrigin`, and the test asserts origin's tip equals it and that the hook ran.
- **F4–F8 (LOW/NIT), fixed.**
  The new and changed comments in `destroy.go`, `remoteleftover.go`, `push.go`'s `DeleteRemoteBranchLeased`, the new test files and `add.go` now use one sentence per line and break only at clause boundaries, per CONSTRAINTS.md's Comment Line-Break Convention.
  The one-line edits inside the legacy fixed-column paragraphs of the `push.go` and `destroyremote_integration_test.go` headers are left unwrapped, because the convention bars rewrapping untouched lines.
- **F9 (NIT), fixed.**
  Every broken-origin `SkipPush` call carries a comment saying that it skips the pre-flight probes and that step 11's warp push ignores it.
- **F10 (NIT), fixed.**
  The `destroyremote_integration_test.go` header says "every case".

## Substrate

`go build ./...` and `go vet -tags integration` over the three packages pass.
`go test` passes for `cmd/lyx`, `internal/gitrepo`, `internal/fabricengine` and `internal/fabriccli`.
`go test -tags integration` passes for `internal/gitrepo` and `internal/fabricengine`.
In `internal/fabriccli`, `TestRunCLI_CleanupRemoteFailureExitsNonZero` fails at Cleanup's archive-tag push, which this diff does not touch.
The round-1 report found the same test failing identically on the merge base.

## Per-card mechanical checks

The plan has no `Rename` or `Delete` group.
Every `Create` target exists: `DeleteRemoteBranchLeased`, `ownedPairWeftBranch`, `dirtyArchivedOnRemote`, `remoteleftover.go`, `ErrRemoteLeftover`, `weftLeftover`, `probeWeftLeftover`, and both new integration test files.
`DeleteRemoteBranchLeased` is pinned in `cmd/lyx/gitrepoboundary_test.go`, and that test passes.

## Code read

The step-12 replacement uses the gate with ownership, archived-on-remote dirtiness and a lease on the probed tip.
Each failure path rolls back through `rollbackAdd`.
`remote_branch_deleted` is recorded only after the leased delete succeeds.
The probes fail in the refusing direction:
a commit missing locally counts as not an ancestor, and a branch that moves between `ls-remote` and fetch leaves the probed tip unverifiable, which refuses.
`pushWeftBranch` already required a weft `origin` without `SkipPush`, so the new weft probe breaks no configuration that worked before.
