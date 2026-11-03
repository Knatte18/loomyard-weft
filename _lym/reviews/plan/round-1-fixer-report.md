# Fixer report — plan, round 1

Every finding's premise held against the code; all thirteen are fixed.
`lym webster validate` reports only `plan-unapproved`, the approval gate.

## Changes

- **F1** (fixed as proposed): card 10 keeps only the "no path reads gone" row in the `tier0` `TestTaskWorktree_AbsentPairIsAnAnswerAndANamedRefusal`.
  The plain-file, empty-directory and resolving-worktree rows move to the `tier1` `TestBattenIntegration_GoneTaskWorktree` as direct calls of `taskWorktreeGone`.
- **F2** (fixed as proposed): card 4 calls `refuseAdoptLocalCheckedOut` under `AdoptOnly` whenever `AdoptLocal` is set and the weft branch exists locally.
  It adds a `TestAdd_AdoptOnly` row with the task branch only on origin and the weft branch checked out elsewhere, expecting `*ErrAdoptLocalCheckedOut` with `Records` true.
- **F3** (fixed as proposed): when card 4 reuses a records worktree and owes a fast-forward, it fetches origin's tip by SHA and runs `git merge --ff-only` inside that worktree instead of fetching into `refs/heads/<weft>`.
  It adds the behind-by-one row.
  The case of a dirty reused worktree that the self-check raised is deferred; see below.
- **F4** (fixed as proposed): card 6's step 1 fetches a differing origin tip by SHA right after reading it, and step 3 no longer fetches.
- **F5** (fixed as proposed): card 8's `Cleanup` re-reads the live warp worktrees under the slug's lock.
  `cleanupRemoteWarp` re-reads the checked-out branches under the lock and keeps a candidate with a live-pair `Reason`, and any other lock error there fills `Error`.
  It adds the `afterEnumerate` row.
- **F6** (fixed differently in placement): card 13 edits `battenCLI.preflightRunOwner` and adds `BattenFollowUp` to an `ownerPreflight` error that names `lym fabric remove`.
  The follow-up replaces the resume's trailing `; then re-run lym batten run <slug>` instead of being appended after it (self-check, fix-proposal F6).
  The test is a new `TestPreflightRunOwner_RemoveFollowUp` rather than a row of `TestOwnerPreflight_GoneTaskWorktree`, because that test drives `ownerPreflight` and only `preflightRunOwner` reads the status the follow-up needs.
  The overview's `d3-sites-outside-fabric` decision names the new site.
- **F7** (fixed as proposed): card 1 creates `PushWithRetryReasons`, re-signs `retryablePushRefusal` with `extraReasons` and keeps `PushWithRetry` with no extra reasons.
  It drops `Repo.PushRebaseFree` from its Edit list, since that push no longer changes.
  A test row asserts that `PushWithRetry` alone does not retry the line.
  Card 6 uses the new entry point for the archive-tag push, passing `missing necessary objects`, and lists it under Uses.
  The overview's card index and card 1's commit title now name caller-named reasons.
- **F8** (fixed as proposed): card 13 re-signs `removeFields` with `warning string` and lists `runRemoveWithFlag` under Edit, which passes `res.Warning`.
- **F9** (fixed as proposed, wording widened): card 10 moves `taskWorktreeLocation` to Edit and adds `presentTaskWorktreePath`.
  The refusal names `lym batten run <slug>` and then repeating the refused command, so it also serves `lym batten cancel` and goto (self-check, fix-proposal F9).
  Both doc comments are rewritten, and the test's pinned assertions are replaced.
- **F10** (fixed with the review's second option): when an exclusive holder has the lock, `PushPairAnchored` returns an error wrapping `ErrPairPushRunning`.
  `commitRecordsVerb` and `addWeftVerbs`' `lym fabric push` read that error as a skip, an ok envelope with `push_skipped`.
  The landing's `PushTask` returns the error, so the landing never reads a skipped push as pushed.
  The callers are added to card 5's Edit list, with a `TestCommitRecordsVerb_Success` row for D7's commit-records criterion.
  `internal/fabriccli/doc.go` gains `push_skipped` (self-check, fix-consistency).
  The plan gate's `doc-go-unlisted` finding then showed card 5 changes `internal/loomcli` without covering its package doc. Card 5 now lists `internal/loomcli/doc.go` under Extends, and its Intent adds a sentence on commit-records' skip envelope.
- **F11** (fixed differently in one row): card 5 drops `PushAnchored` from Edit and from its test bullets.
  The proposal said to drop the prime records-side row; I rewrote it over `PushPairAnchored` at the prime instead.
  Dropping it would leave untested the card's claim that the prime's records push takes no teardown lock, and `PushPairAnchored` pushes the records side at the prime (`pushanchored.go:126`).
- **F12** (fixed as proposed): card 3's `config.lock` row sets `branch.<weft>.remote` and `branch.<weft>.merge` before planting the lock.
- **F13** (fixed as proposed):
  - Card 1 lists `ErrLeaseRejected` under Edit and updates the `pushretry.go` header and the lease bullet in `gitrepo/doc.go`.
  - Card 2 adds both primitives to the `push.go` header.
  - Card 6 lists `Topology.Remove`, `archiveTagPushDelays` and `pattern/PATTERN-fabric-git.md` under Edit and rewrites the docs it falsifies, the `archive.go` header among them.
  - Card 8 moves `RemoteWarpBranchEntry` to Edit and updates its `Reason` and `Error` docs.

## Self-check

- fix-consistency, LOW: `push_skipped` missing from the fabriccli envelope doc. Fixed as proposed (card 5 Extends `internal/fabriccli/doc.go`).
- fix-consistency, LOW: card 1's index line still named the old retry. Fixed as proposed.
- fix-claims: no findings.
- fix-proposal, MEDIUM (F6): the follow-up was appended after the resume's re-run. Fixed as proposed: the follow-up replaces the trailing reentry, and the test rows assert that the reentry is gone.
- fix-proposal, MEDIUM (F3): `merge --ff-only` in a dirty reused records worktree fails with an answer outside D2's closed set. Deferred; see below.
- fix-proposal, LOW (F9): the refusal is also reached from cancel and goto. Fixed as proposed: the refusal names repeating the refused command after the resume.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

- **Self-check fix-proposal F3 (MEDIUM): a reused records worktree with uncommitted records in a path the fast-forward rewrites.**
  `git merge --ff-only` refuses there, and the error is transient under card 12, so every pass retries it.
  Every sound answer needs a typed refusal outside D2's closed set.
  `_lym/discussion/decision-record.md:60-68` lists that set member by member and says that "any refusal outside the closed set" is transient, so adding a member changes the decision record.
  That is a decision for the operator or the parent, not for this round.
  The case is narrow: origin's weft can only be ahead of the local copy when another clone pushed the slug's records, and card 10's first reconcile refuses an entry owned elsewhere.
  A later adopt back to this machine can still reach it.
