---
verdict: APPROVED
rationale: "All eight round-1 findings landed in their own commits (123bf636d..fb6d0dcd8), the worktree is clean, and the plan's whole verify block (build, vet, go test, the -tags integration package list, vet -tags smoke) passes on HEAD; F1's .git-entry stat keeps card 6's one-error contract without a git spawn, and nothing new surfaced."
---

The review range is `git diff $(git merge-base main HEAD)..HEAD`, with `product.parent` read as `main` from `_lyx/shed/agent-naming/status.json`.
The round-1 report found the range correctly and left the uncommitted `hubgeom.go` edit out of scope.
That edit no longer exists: `git status` is clean, because `123bf636d` replaced it.

## Findings, checked against HEAD

- **F1 (MEDIUM), fixed in `123bf636d`.**
  `ReedGeometry` now returns an error when it cannot tell the prime from a task worktree, so a task worktree never gets a prime's geometry.
  This keeps card 6's contract: "A `PrimeName` failure is the function's only error."
  The fix tells the prime by statting `<worktree>/.git`, where card 6 named `fabricengine.PrimeName`.
  I accept that change of mechanism.
  It derives the same fact, the main worktree against a linked one, and only the stat keeps the untagged wiring tests within the Test Tier Purity Invariant.
  `PrimeName` would spawn `git worktree list` on every hub wiring path.
  No existing helper reads a `.git` entry without spawning git (`isRegisteredLinkedWorktreeIn` goes through `List`), so the new `isPrimeWorktree` duplicates nothing.
  `TestIsPrimeWorktree` covers each branch of the rule.
  The three Tier 1 `hubLocation` helpers create only a `.git` directory, so they stay offline.
  `ReedGeometry`'s doc and `hubgeom/doc.go` both name the stat and the error.
- **F2, fixed in `fb6d0dcd8`.**
  Spot checks of `codebinding.go`, `strand.go`'s `AddSpec`, `seed.go`'s `Parent` and the `reedgeom_integration_test.go` header show one sentence per line, with breaks only at clause boundaries.
- **F3, fixed in `c841675d9`.**
  `LoomDriverStrandName`, `statusStrandDisplayName` and `resolveStatusStrandAction` now say reed refuses the add.
  They agree with `driverSpec`.
- **F4, fixed in `eeda57ca4`.**
  `paneIsLive` is gone.
  `repairNames` and `directoryRows` both test liveness through `aliveIDSet`, and each keeps the empty-`PaneID` guard.
- **F5, F6, F7 and F8, fixed in `c05efc4e6`, `eb6988638`, `5e8a40321` and `827a4e9f8`.**
  Each diff makes exactly the change its finding asked for.

## Evidence

On HEAD, in the worktree, each of these exits 0:

- `go build ./...`, `go vet ./...` and `go test ./...`.
- `go test -tags integration` over the plan's full package list.
- `go vet -tags smoke ./...`.

## Mechanical checks

The plan has no `Rename` or `Delete` group.
The report confirmed that card 7's in-Edit deletions have no remaining callers.
F4's removal of `paneIsLive` leaves no reference to it.

The report has no `## Focus departures` section, so there is no departure to ratify or reject.
