# Merge main into the beam adopt-build-resplit

Brings the parent branch `main` into the beam `adopt-build-resplit`, as a true merge with no pull request.
The diff is `main`'s own work carried in; this description covers only what the merge itself decided.

## Conflicts

The merge conflicted in three files, each settled by keeping both sides' intent:

- `internal/beamcli/pr_test.go`: the beam's `PushThatHandedTheRunOverIsReturnedWithoutARerun` case keeps its place in `TestPROpen`.
  Its expectation moves to `wantNoErr: []string{"again"}`, the field `main` renamed from `wantNot`.
- `internal/lymdirs/doc.go`: the package comment keeps `main`'s `HandOverDirName` line.
  It also keeps the beam's wider list of exceptions under the tracked `LymDirName`: `UsageDirName`, `RaddleDirName` and `WeaverDirName`.
- `internal/reedengine/bindings_integration_test.go`: settled by hand to carry both sides.

## After the merge

One commit follows the merge, `fix: break overview comments at sentence boundaries`.
It re-breaks comments in `internal/heddleengine/overview.go` and `overview_test.go` at sentence boundaries, with no change in behavior.
The merged tree carried comments that broke that rule.

## To check by hand

- Read the settlement of `internal/reedengine/bindings_integration_test.go` in the merge commit, the one conflict not described above.
