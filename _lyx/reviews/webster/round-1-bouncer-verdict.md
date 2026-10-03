---
verdict: BLOCKING
rationale: "F1-F6 are fixed and build, vet, unit and integration tests pass at 44214954f, but seven new or changed Go comments the review's F5 list missed still join independent clauses with a semicolon, against CONSTRAINTS.md's Comment Line-Break Convention, e.g. mergestage.go:131, cardgate.go:2, verifygatereport.go:64"
---

## Range

`_lyx/shed/verify-publish-hardening/status.json` names `product.parent: main`, and no `_lyx/loom/rework/` directory exists.
The subject is therefore `git diff $(git merge-base main HEAD)..HEAD`, which is `5805c58fb..44214954f`, cards 1–20 plus the six fixer commits.
The review used the same range.

## The round-1 findings

Each fix was checked against its own finding.

- **F1** (`4ea72a75d`): `MergeUntrackedFiles` now reads `f.warp.UntrackedFiles()` alone and drops the `lyxcwd` and merge-geometry resolution.
  `unifyConflictPaths` and `resolveMergeGeometry` still have their `MergeIn` callers.
  The integration test writes an untracked weft file and asserts that the list omits it and the call succeeds.
- **F2** (`0d53f6f1f`): after a pass, `Verify` re-reads `HEAD^{tree}` and writes the record only when it is unchanged.
  `TestVerify_CommitDuringRunLeavesNoRecord` pins this.
  A moved HEAD still returns `StatusPassed` without a record, which is the fix the review asked for.
- **F3** (`6a9f4288e`): `DirtyPaths` runs `git status --porcelain -z`, and `parsePorcelainZ` skips the source record that follows an `R` or `C` entry.
  A tier-1 table test and a real-git test cover special characters, an arrow inside a name, and renames.
- **F4** (`b1c47b14b`): a grep outside `_lyx` for `integration fork`, `integration report`, `ParseIntegrationReport` and `integrationFix` finds only intentional hits.
  These are the negative assertions in `claims_test.go` and `template_test.go`, and the `state_test.go` load-compatibility fixture.
- **F5** (`9fedd120b`): every site the review listed is fixed (see below for what the list missed).
- **F6** (`44214954f`): the `config.go` knob comment now says a zero `verify_gate_attempts` turns the must-pass verify gate off.

`go build ./...`, `go vet -tags smoke ./...` and `go test ./...` pass on HEAD.
So does `go test -tags integration` over verifytree, fabricengine, mergeresolve, websterengine, landingshed, webstercli, friction and loomshed.

## Per-card mechanical check

Card 17's and card 20's `Delete` groups have no remaining callers.
A grep outside `_lyx` for `CheckoutDetached` and `RestoreBranch` is empty.
`gitrepoPinnedRunBoundMethods` shrank with them, and the build passes.
No card carries a `Rename` group.

## Why this blocks

The rubric's `Also flag` section requires new and changed comments to follow the target repository's constraints document.
CONSTRAINTS.md's Comment Line-Break Convention requires one sentence per line, with a further break at an independent-clause boundary.
The review's F5 list was incomplete.
These new or changed comments still join two independent clauses with a semicolon on one line:

- `internal/fabricengine/mergestage.go:131`: "Untracked files are never staged; MergeUntrackedFiles lists them…" (new in card 4).
- `internal/fabricengine/mutation.go:83`: the new `KindMergeTrackedStaged` doc joins "mid-merge; Target is the warp checkout path." and also starts a second sentence on that line.
- `internal/websterengine/cardgate.go:2`: "Go derives the command and the fork runs it; Go never runs it, so…" (new file).
- `internal/websterengine/verifygatereport.go:64` and `internal/websterengine/doc.go:401`: "The hint claims only that a card touched a failing package; Merriam judges it with the plan in hand." (new).
- `internal/websterengine/runlevel.go:724`: "A done whose verify gate did not pass ends stuck; Master's own stuck keeps its own reason." (changed line).
- `internal/websterengine/runlevel_test.go:113`: "…on this helper; a test that needs one calls appendIntegrationVerify…" (changed line).

The fix is mechanical: break each line at the semicolon.

Edits inside legacy fixed-column paragraphs are deliberately not counted.
In those paragraphs, a sentence boundary falls mid-line only because of the existing wrap, and fixing it would mean rewrapping untouched lines, which the convention forbids.
The excluded sites are `fabricengine/doc.go`'s "that is still recoverable. The weft's own detachment…", `gitrepo/doc.go`'s two edited lines, `mergeresolve/spec.go:21`, `websterengine/render.go`'s friction-role paragraph and `runlevel.go`'s `accumulatedCardSHAs` paragraph.

## Focus departures

The round-1 review carries no `## Focus departures` section, so there is nothing to ratify or reject.
Its directive verdicts on verify-directory placement, `NewVerifyGate` arrival order, gate parity, landing order, merge-stage staging, shuttle and launch, and card 12 were spot-checked against the code and are accepted.
