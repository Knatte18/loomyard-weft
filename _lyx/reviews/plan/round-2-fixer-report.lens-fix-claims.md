```yaml
lens: fix-claims
outcome: reported
```

Claims checked against HEAD. One finding.

- LOW — card 15 Intent, the new `UpstreamRunUnfinished` sentence ("reads the pair's own run status at the tree of its weft's fetched upstream tip"), and card 17's matching clause ("the run status is judged from the weft's fetched upstream tip through `UpstreamRunUnfinished`, never the checked-out copy").
  These sentences say the owned-elsewhere path reads only the pair's own loom status, at `TreeRunFileRels(slug)` on the pair's weft.
  The code also stores a beam's batten run status somewhere else: on the prime's weft, as `shedrun.StatusFile(prime, slug)` (`internal/fabricengine/replay.go:283`, `internal/beamcli/dobbydeps.go:310`).
  Card 17 still says only "its run status", so it does not say whether the owned-elsewhere refusal still reads the batten status on the prime's weft.
  That copy is not stopped by this pair's marker change, so it is not stale.
  As written, "never the checked-out copy" can be read as dropping that check.
  Evidence: the `readRunStatusRows(shedrun.StatusFile(primeLocation, slug))` call in `finalizeEnteredRead`.

These claims hold:
- `replayWeft` gates with `ownedWarpCheckout(worktree)`, which resolves through `isAnyWorktreeOf`, so the main checkout is included (`destroy.go:604-608`, `replay.go:437`).
- `ClearPushFailure` is a tolerant `os.Remove` in `failedpush.go`, and `failedpush.go` and `mergestate.go` each have their own allowlist entry.
- `judgeOrigin` refuses an owned-elsewhere slug with a present worktree and a reachable origin (`cancel.go:396-405`).
- The legacy `self` fallback applies when the slug's directory is absent (`resolve.go:28-45`).
- `TreeRunFileRels` returns paths under `_lyx/shed/<id>/`, relative to the anchor, so joining them under `treeAnchor` is right.
- Card 17's order (BeginAnchorChange, pass wait, parent lock, dobby end, sink close, Apply) matches the claim that those errors come after the record is written.
- On Linux, renaming a directory onto a non-empty one fails with ENOTEMPTY, which supports the restore-failure hook row.

Tool calls:
- Read the `pathOwnershipWarpCheckout` resolution in `destroy.go` and how the prime weft is created. My context showed the `ownedWarpCheckout` constructor but not what its resolver accepts.
