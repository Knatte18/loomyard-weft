# Fixer report — code review round 1

All ten findings fixed. Tier 1 gate on `./internal/fabricengine ./internal/battencli ./internal/pairteardown` passed with 0 failed, and again on `./internal/fabricengine` after the self-check fixes. Nothing pushed.

## Changes

- **F1** (`pairbranch_integration_test.go`): `TestBranchPrefixChange_PairWithNothingLeftLocally` makes a pair under the empty prefix, removes it locally with both branches left on origin, then sets `hanf/`.
  Its subtests assert that `Remove` with remote deletes the bare-slug task and records branches on origin and reports the bare `SiblingBranch`, that `DiscardTaskBranch` deletes the bare-slug task branch on origin, and a table over which branches origin holds.
  The table covers `RemoveRefusal`, `PairTaskBranchOrOrigin` and the network-free `PairTaskBranch`, including the records-only and task-only rows for `originPairBranch`.
- **F2** (`pairbranch_integration_test.go`): `TestPairTaskBranch_ReadsThePairItself` has rows for a task worktree on an unrelated branch, a detached task worktree, only the records worktree standing, only local bare branches, only the local records branch, a prefixed and a bare branch both local, and nothing left.
- **F3**: `TestTaskBranchSlugs` gets a row with `branch_prefix` set (bare and prefixed branches both listed).
  `TestCleanupRemoteWarp_BranchPrefixSetOverBareBranches` pins a bare branch with weft evidence as managed and deleted, and one without as not fabric-managed.
  `TestCleanup_PrefixSetOverBareSlugOrphan` pins the orphan sweep's archive tag under the bare slug.
  `TestAdd_FromParentPairMadeBeforeBranchPrefix` pins a bare-slug `--parent` accepted and an unrelated branch refused.
- **F4** (`remove.go`): `pairState.localPartRemains` is the one local-remnant test, used by `refuseWhenNothingRemains` and `removePair`.
  `RemoveRefusal` now wraps an unexported `refuseRemoval` that also returns the name origin holds the pair under, so a `Remove` asks origin at most once.
- **F5** (`pairbranch.go`, `pullrequest.go`, `readopt.go`): `PairTaskBranchOrOrigin` returns the `PairTaskBranch` answer unless the local read fell through to its last fallback, then origin's name.
  `closePairPullRequest` and `hubTaskBranch` use it. `PairTaskBranch` stays network-free.
- **F6** (`pairbranch.go`): the records-worktree step is skipped when `RecordsRepoRoot` fails, falling through to the local-branch step.
- **F7**: a `battencli` re-adoption subtest asserts `TaskBranch` and `TaskBranchGone` answer for the bare slug under a prefix.
  A new `pairteardown` test, with `export_test.go`, asserts the gone-worktree pull request warning names the bare slug.
- **F8** (`template.yaml`): the `branch_prefix` description says each new pair gets the prefix, a pair keeps its branch name, the value can change live, and empty means branch equals slug.
- **F9** (`destructivegaps_integration_test.go`): the comment names `ownedManagedBranch(l)`.
- **F10** (`remoteteardown.go`): `WeftBranch` and `WarpBranch` each have their own comment.

Fixed differently from the proposal:
- **F4**: the proposal said to return the origin answer from "an unexported variant of RemoveRefusal that removePair calls".
  I did that, and `removePair` still re-inspects the pair and applies the name only when no local part remains, so a part that appeared between the two reads is not overridden.
- **F1**: the proposed origin-only-records row is expressed through the table above, because `originPairBranch` is reached through `RemoveRefusal` and `PairTaskBranchOrOrigin`.

Commits, one per finding or shared file set: F8, F9, F10, F4, F5+F6 (both change `pairbranch.go`), F1+F2 (one test file), F3, F7, then three self-check commits.

## Self-check

Four forks returned reports, saved as `round-1-fixer-report.lens-<lens>.md`.

- fix-consistency, LOW: `doc.go` does not name `PairTaskBranchOrOrigin`. Fixed: `doc.go` now names it.
- fix-consistency, LOW: `doc.go` does not mention the skipped records step. Fixed in the same paragraph.
- fix-affected-set, LOW: `deleteLeftoverUnderLock`'s slug derivation is not pinned, since the bare lock slug equals the branch. Fixed: `TestCleanupRemoteWarp_BranchPrefixSetTakesTheBareSlugsLock`.
- fix-proposal, MEDIUM: F2's rows cannot tell the records-worktree step from the local-branch step. Fixed with my own change: a row where only the records step answers before a local prefixed branch (`RecordsWorktreeStepAnswersBeforeLocalBranches`).
  The fork's suggested row, a records worktree on `hanf/pt-pair-weft`, would also be answered by the local-branch step, so it does not discriminate.
- fix-proposal, LOW: nothing tests that a launcher-only pair makes no `ls-remote` or that an origin-only `Remove` asks once. Deferred, see below.
- fix-code-correctness, LOW: the `PairTaskBranchOrOrigin` doc claims no network call for any local remnant. Fixed: the doc says a pair the local read names makes no network call.

## Disputed

None.

## Deferred

- **F6 test row**: no input makes `RecordsRepoRoot` fail while `PairTaskBranch`'s earlier `List` calls succeed, because it fails only when `List` on the anchor path fails, and no seam exists to inject that failure. The code change is made; the F2 table has no row for it.
- **Self-check, F4 call count**: pinning that a `Remove` makes at most one `ls-remote` needs a production seam that observes `Remove`'s remote calls, which the review did not ask for. It is an operator decision whether to add one.
