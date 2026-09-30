---
verdict: APPROVED
rationale: "All three round-1 findings are fixed in the current plan. F1: card 2 now forks through a new unexported function in forkrecords.go and leaves createWeftWorktree unchanged for Reconcile's createDormantWeftForRawWarp. F2: the write-out uses checkout-index -a -u and read-tree -m HEAD. F3: the deps.go header edit is gone. No new rubric finding survives."
---

## Round-1 findings against the current plan

- **F1 (BLOCKING), card 2: resolved.**
  Card 2's step 1 replaces `Add`'s step-8 call to `createWeftWorktree` (`internal/fabricengine/add.go:202`) with a new unexported function in the created file `forkrecords.go`.
  It lists `containedWorktreeAdd` (`destroy.go:1241`) under Uses and records `KindWorktreeCreated`/`KindBranchCreated` right after the fork, as `createWeftWorktree` does at `weftwiring.go:132-133`.
  `createWeftWorktree` is no longer a target, and `reconcile.go` stays out of scope.
  The Intent explains why (the Reconcile caller never commits, and the decision record limits the drop to `Add`), and the ImpactSummary now says Reconcile is unchanged.
- **F2 (LOW), card 2: resolved.**
  Step 2 now runs `git checkout-index -a -u` in both branches and restores the root with `git read-tree -m HEAD`, and it says why: the index keeps stat data.
- **F3 (LOW), card 3: resolved.**
  The doc-update sentence now covers only the two `Call` doc comments, `PrimeLock`'s doc comment and `doc.go`, and every one of those sits under a card-3 target.

## Other checks

- Decision record fidelity:
  Decisions 1, 2, 5 and 6 and the four Constraints on the drop (Shed Run-Directory, no records on disk, Destruction Chokepoint via the no-checkout fork, Fabric Git through `CommitWeftPaths`) are all in card 2.
  Decision 4 is card 1.
  Decisions 7 and 8 and the transient-mark rule are card 3.
  The docs and `CONSTRAINTS.md` criteria are card 5, and each package doc update belongs to the card that owns the package (1, 2 and 3).
  No card adds scope the record does not allow.
- Verify coverage: `go test ./...` plus `-tags integration` over `shedrun`, `fabricengine`, `battenshed` and `battencli` runs every package any card targets.
- Seams exist as the plan names them: `PathTracked`, `ScopedPathspec`, `CommitWeftPaths` (it returns early on `SkipGit`, which matches card 2's `SkipGit` sentence), `OriginRecordRel`, `waitOrCancel`, `cancelErr`, `fakePrimeLock`, and `paths_test.go`'s `syntheticLocation` for card 1's `SeedRel`/`StatusRel` assertions.
- The new index-only `git rm -r --cached` call in `forkrecords.go` matches none of `cmd/lyx/destructiveguard_test.go`'s nine banned tokens.
- Granularity: card 4 is the `battencli` acceptance pair, in a different package from card 2's `fabricengine` test, so it can be reviewed on its own; card 1's constructor can be tested on its own.

## Focus departures

The round-1 report has no `## Focus departures` section, so there is nothing to ratify.
