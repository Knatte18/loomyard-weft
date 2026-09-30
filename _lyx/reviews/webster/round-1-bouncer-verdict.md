---
verdict: APPROVED
rationale: "Range main..HEAD (merge base dcc7d7dcd) implements all five cards; the round-1 review's three LOW/NIT findings are fixed in 1d2702a97, 292a14e7e and 1ad5ac7ed, and go build, go vet and the plan's unit and integration verify suites pass at HEAD."
---

The review range comes from `_lyx/shed/batten-records-leak-into-pairs/status.json`, where `product.parent` is `main`.
The review covered `git diff $(git merge-base main HEAD)..HEAD`: nine commits, `fc1db0884` through `1ad5ac7ed`.

## Plan conformance

- **Card 1.**
  `shedrun.RunsRootRel` returns `filepath.Join(lyxdirs.LyxDirName, shedDirName)`, and it has its doc comment, the file-header update and the `doc.go` sentence.
  `TestRunsRootRel` pins that `SeedRel` and `StatusRel` both lie strictly under it.
- **Card 2.**
  `forkrecords.go` runs the no-checkout sequence the card specifies: `worktree add --no-checkout`, then `read-tree HEAD`, then `PathTracked`, then `rm -r --cached` only when the root is tracked, then `checkout-index -a -u`, then `read-tree -m HEAD`.
  It records `KindWorktreeCreated`/`KindBranchCreated` right after the fork.
  `createWeftWorktree` is unchanged, so `Reconcile` still uses the plain fork.
  Step 10c adds `RunsRootRel()` to the commit paths only when `runRecordsTracked` is true, and the adopt path leaves that flag false.
  The integration file covers every listed criterion, and since round 1 it also covers a subpath-anchored hub.
- **Card 3.**
  `primelockwait.go` holds both constants and `acquirePrimeLock`.
  The wait is attempt-counted: 300 sleeps and 301 attempts.
  It logs one info line on the first contended attempt and checks cancellation after contention and after each sleep.
  An `Acquire` error defers to `cancelErr` before returning the hard error.
  `PrimeLock.Sleep` resolves nil to `waitOrCancel`.
  `wire.go`, the only production construction, is unchanged.
  Every wait case the card lists has a test for both `Worktree-Create` and `Worktree-Teardown`.
- **Card 4.**
  Both batten integration tests exist.
  The Seed-Child test now asserts that `Worktree-Create` ends `Done` before it steps `Seed-Child`.
- **Card 5.**
  The `docs/overview.md` batten bullet and the new Shed Run-Directory Invariant bullet in `CONSTRAINTS.md` say what the card asks.

## Also-flag dimensions

- **Comment convention.**
  After `1d2702a97`, every new or changed comment in the diff has one sentence per line, with further breaks only at clause boundaries.
  This follows `CONSTRAINTS.md`'s Comment Line-Break Convention.
  The fixed-column-wrapped lines left in `create.go`, `teardown.go` and `deps.go` are untouched legacy text, which the convention exempts.
- **Per-card mechanical checks.**
  - The Edit groups' blast radius holds: `PrimeLock` has one non-test construction and it compiles unchanged, and every `Add` caller's tests pass.
  - The Create groups duplicate nothing already present: `shedrun` had no root constructor, and there was no no-checkout fork helper.
  - No card carries a Rename or Delete group.
- **Fabric Vocabulary Invariant.**
  No added text in `battenshed`, `shedrun` or `battencli` names a fabric side.

## Round-1 findings

- **F1 (comment breaks):** fixed and verified in the diff.
- **F2 (no subpath-anchor coverage):** fixed.
  `TestAdd_DropsParentRunRecords_SubpathAnchor` runs on a `"backend"`-anchored hub.
- **F3 (Seed-Child test ignored the Worktree-Create outcome):** fixed.

The review report has no `## Focus departures` section, so there are no departures to ratify.
