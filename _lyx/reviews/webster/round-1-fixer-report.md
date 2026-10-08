# Webster round 1 — fixer report

Every finding was validated against the code and the plan and holds; all six are fixed, one commit each.

## Fixed

- **F1** (`95a9be0fe`): `createGitWorktree`'s doc joins its compound object onto one line.
  `rollbackAdd`'s doc splits the `warp` sentence into two sentences: the proof "lets the gate delete the local branch", and "It also carries what step 11 did about origin."
- **F2** (`0b75d9cb2`): `PushPairAnchored`'s doc names the failing side "code or records", matching `pushSide("code", …)`/`pushSide("records", …)`.
- **F3** (`1d18903ea`): `internal/fabricengine/doc.go` names the `Fabric` handle's fields `code *gitrepo.Repo` and `records *gitrepo.Repo`.
  The touched sentence now uses semantic line breaks.
- **F4** (`4624c0a48`, `3f58ca928`): `TestWeftBranchName*` became `TestRecordsBranchName*`, `TestCommitWeftPaths_*` became `TestCommitRecordsPaths_*`, `TestResetPairWarp_*` became `TestResetPairCode_*`, `TestWarpLyxLinkMethods` became `TestCodeLyxLinkMethods` (with its `wantWarpLyxLink` field becoming `wantCodeLyxLink`), and `mustWeftRepoRoot` became `mustRecordsRepoRoot`, along with every call site and doc line that cites them.
  File names are unchanged, as Card 5 says.
  `WarpLyxLinkHere` and `WeftWarpSlug` are not renamed functions under any card, so their spellings stay.
  The follow-up commit re-breaks `origin_integration_test.go`'s header, which the helper rename touched, at its semicolon so `lyx loom lint-comments` passes.
- **F5** (`1dbc9804d`): `internal/fabricengine` in `cmd/lyx/testdata/test-budget.yaml` is 626, the package's top-level test count at HEAD excluding `TestMain`.
- **F6** (`eeb358937`): `TestGate_ZeroValueDeclarationsAreRefusals` in `destroy_test.go` gains four rows, with no new top-level test:
  - `ownedCreatedBranch` refusing a foreign branch name;
  - `ownedCreatedBranch` refusing an empty proof;
  - `dirtyPushedByThisCall` refusing an empty lease on a remote delete;
  - `dirtyPushedByThisCall` refusing a local delete.

  Disabling the matching guards in `destroy.go` makes the first three rows fail.
  The local-delete row still refuses with its explicit check removed, because `checkBranchDirtiness` refuses any unhandled kind.

## Also changed

- `1ba511ae5`: gofmt on `warpbinding_reconcile_integration_test.go`.
  Card 4's `CodeBare`/`RecordsBare` field renames had left the `clonedHubFixture` struct and its literal misaligned.
  `internal/fabricengine/pull.go` is also not gofmt-clean, but it was already that way at the merge base, so it is untouched.

## Verification

At HEAD these all pass:
- `go build ./...`
- `go vet` under the default, `integration`, `tmux` and `llm` tags
- `go test ./...`
- `go test -tags integration ./...`
- `lyx loom lint-comments --base dc30e1a98`

## Disputed

None.

## Deferred

None.
