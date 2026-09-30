---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: internal/battencli/wire.go:202
    summary: teardownRefusal names commit-records for every ErrPairSiblingDirty, but the refusal fires on any uncommitted content in the sibling, so content outside the records pathspec loops the operator through the same remedy, and docs/overview.md claims a distinction the code cannot make
  - id: F2
    severity: LOW
    location: internal/battencli/wire.go:203
    summary: the ErrArchiveFailed arm tells the operator to make the remote reachable, which is the wrong remedy for a weft-repo resolution failure, a clashing archive tag or a push the remote rejected
  - id: F3
    severity: LOW
    location: internal/battenshed/innerrun.go:314
    summary: an unparseable done-seen marker is a hard error on every Call and no resume clears it, so a torn marker write wedges the run with no remedy named
  - id: F4
    severity: LOW
    location: docs/overview.md:404
    summary: the batten status sentence still says only the still-running self-bounce appends history under a 1440-bounce budget, although the awaiting and driver-exit waits now append unbudgeted entries too
  - id: F5
    severity: LOW
    location: internal/battencli/records_integration_test.go:143
    summary: card 16's end-to-end test never asserts the run ends done past Worktree-Teardown, one of the four outcomes the card lists
  - id: F6
    severity: LOW
    location: internal/battenshed/innerrun.go:136
    summary: many new and changed Go comments across the diff are fixed-column wrapped mid-sentence, against CONSTRAINTS.md's Comment Line-Break Convention
  - id: F7
    severity: NIT
    location: contracts/recipes/batten-recipe.yaml:13
    summary: the recipe header comment narrates the change (Row names and max_bounces are unchanged) rather than describing current state
---

# Webster-Review round 1

Review range: `product.parent` is `main` (`_lyx/shed/batten-hub-ready/status.json`), merge base `083e992cf`, 17 commits, 65 files.
The plan's `verify:` block passes in full on HEAD `a25c57c0d`: `go build ./...`, the untagged and `integration`-tagged test runs over every listed package, and `go vet -tags smoke`.

## Focus directive verdicts

- **Commit a25c57c0d (no card).**
  `ErrArchiveFailed` is wrapped onto every `archiveWeftTip` error by a deferred `%w: %w`, the named-return shadowing is sound (every explicit `return` assigns the named `err`), and the sentinel is tested in `wire_test.go` and by `TestBattenIntegration_Teardown_UnreachableRemoteHaltsBeforeRemovalResumably`.
  The archive runs in `Remove` before `removePortal`, so "the pair was left in place" holds for every `archiveWeftTip` failure; CONSTRAINTS.md's Fabric Git bullet and docs/overview.md's new "A failed archive push halts the row the same way before anything is removed" both agree with the code.
  The remedy text is wrong for some causes (F2).
  The reworded `incompletePairRemedy` and its doc comment match `remove.go`: `deleteWarpBranch` runs after both worktree removals and deletes the task branch when the destructive gate finds its work on another ref or landed on the recorded parent, which a fresh create's branch always satisfies.
- **ErrPairSiblingDirty mapping.**
  `refuseDirtyWeftWorktree` runs `worktreeDirty(scopeAll, …)`, so any modified or untracked path in the sibling fires it, including paths outside `statusCommitPathspec`.
  Defect, F1.
- **ErrArchiveFailed remedy per cause.**
  "Left in place" holds for all causes; "make the remote reachable" holds only for the `ls-remote`, `fetch` and unreachable-push failures (F2).
- **Card 10's resume via `lyx loom start --no-attach`.**
  Not a defect.
  `seedAndCommitBootstrap` accepts `ErrSeedExists` and never reseeds; `runDriverSpawnAndWait` spawns whenever `mustSpawnDriver(runLockHeld, driverLive)` says so, with no gate on the run's state; the go arm's `halted()` reads `State != running`, so an awaiting run completes the handshake on its first poll, and the llm arm has no handshake.
  This is the same bootstrap the pre-change docs told the operator to run by hand.
  Ordering: `Spawn` → `recordSpawnConfirmed` → approval-acted marker write.
  A failed marker write returns a hard error after a successful spawn, and the resumed Call re-spawns, which the bootstrap tolerates against a live driver.
- **Marker lifecycle.**
  A stale approval-acted marker cannot wedge a later run of the slug: the identity carries `approved_at` in RFC 3339, so a later approval never matches it.
  An unparseable done-seen marker can wedge a run (F3).
- **Card 1's budget arm.**
  Correct.
  `!output.BudgetExempt` short-circuits the comparison, `appendHistory` copies the flag only for `outcome == Stuck`, and `episodeStuckCount` skips exempt entries.
  `TestStep_ExemptStuckAfterBudgetFullySpent`, `TestInnerRun_DoneWaitIsNeitherCountedNorBlockedByTheRunningBudget` and `TestInnerRun_AwaitingWaitIsNotBlockedByTheRunningBudget` all seed N counted Stucks against a budget of N, so they exercise the fully-spent case and not only the counting.
- **Cards 12/13 ordering.**
  Correct.
  `Remove` archives after the slug, prime, existence and both merge refusals and before `removePortal`.
  `RemovePairBranch` archives after its sibling-remnant refusal and before either deletion.
  `Cleanup` archives before `deleteWeftBranch` and skips the branch on failure.
  `nameStrandedPortalTeardown` compares against the post-archive length, so only portal and launcher mutations can trigger the stranded-teardown remedy, and those always grow the record.
  Cleanup's `strings.TrimPrefix(warpBranch, BranchPrefix)` inverts `Remove`'s `BranchPrefix + slug` → `WeftBranchName`, so both tag under the same slug.
- **Per-card mechanical checks.**
  `reviewsHoldFile` and `awaitingChildRemedy` have no surviving reference in code, tests or docs outside `_lyx/`.
  None of the Create targets (`DriveReportsDir`/`Rel`, `LoomDurableDir`/`Rel`, `LoomDriverStrandName`, `ErrNoSession`, `ChildApproval`, `archiveWeftTip`, `KindTagPushed`, `ErrPairSiblingDirty`, `commit-records`) existed at the merge base, and no equivalent existed: fabricengine had no tag helper, reedengine no exported sentinel, and the driver strand name only as loomcli's unexported constant.
  `NewInnerRun` has the one registry caller card 10's ImpactSummary names, and the other ImpactSummaries match the callers the diff touched.
- **Comment convention.**
  Defect, F6.

### [MEDIUM] teardownRefusal names commit-records for dirt the records commit cannot clear

**Location:** `internal/battencli/wire.go:202` (`teardownRefusal`), `docs/overview.md:398`

**Issue:** `ErrPairSiblingDirty` is wrapped by `refuseDirtyWeftWorktree`, which runs `git status --porcelain` over the whole sibling (`scopeAll`).
Any modified or untracked path there fires it: a half-written `_lyx/plan/` card, a `_lyx/discussion/` edit, `_lyx/landing/summary.md`, a stray file.
`teardownRefusal` maps every such refusal to "the task worktree's run records are uncommitted … run `lyx loom commit-records` … then resume".
`commit-records` stages only `statusCommitPathspec` (status file, reviews, `_lyx/loom`, drive reports), so for dirt outside it the operator runs the named remedy, resumes, and gets the identical refusal and remedy again, with no exit named.
docs/overview.md says "other uncommitted content keeps fabric's refusal", which the code cannot deliver: it cannot tell a records path from any other dirty path in the sibling.
Content dirty in the task worktree itself does keep fabric's refusal, but that is a different error.

**Fix:** Keep the commit-records remedy as the first step, and add the way out when it is not enough.
If the resumed teardown refuses again with this reason, what remains uncommitted is not a run record: inspect the pair from the task worktree, commit or remove that content by hand, then resume; still never `--force`.
Reword docs/overview.md to match: an uncommitted change in the pair's sibling names batten's own recovery (`commit-records`, then resume), and names committing or removing the leftover by hand when it is not a run record.
Uncommitted content in the task worktree itself keeps fabric's refusal.
Extend `wire_test.go`'s assertion to the new wording.

### [LOW] The failed-archive remedy assumes an unreachable remote

**Location:** `internal/battencli/wire.go:203`

**Issue:** `archiveWeftTip` wraps `ErrArchiveFailed` onto a `WeftRepoRoot` resolution failure, an existing local tag of the same name at another commit, a `for-each-ref`/`tag` failure, and a push the remote rejects (a protected `refs/tags/archive/*`, or a tag of that name already on the remote at another commit), besides the `ls-remote`/`fetch`/push failures of an unreachable remote.
The refusal tells the operator to "make the remote reachable" for all of them, and for a clashing tag a plain resume fails identically forever.
"The pair was left in place" holds for every cause: the archive runs before `removePortal`.

**Fix:** Name the underlying failure as the thing to fix, with the unreachable remote as the common case: "fix the failure named above (most often an unreachable remote), then resume this run …".

### [LOW] An unparseable done-seen marker wedges the run

**Location:** `internal/battenshed/innerrun.go:314` (`callDone`)

**Issue:** A done-seen marker that exists but does not parse as RFC 3339 returns a hard error on every Call.
Resuming batten re-enters `callDone` and fails the same way, and the error names neither the file nor a remedy.
The producer's own `os.WriteFile` can leave such a marker: a process killed between the create and the write leaves an empty file.
Card 10 makes marker read, write and removal failures hard errors.
A marker that reads fine but holds garbage is not a mechanism failure, and the marker only exists to bound a wait.

**Fix:** Treat an unparseable marker as a fresh first sight: log a warning naming the path and the parse error, overwrite it with `now`, and continue the wait.
Add a unit test.

### [LOW] docs/overview.md's history-cap sentence still describes only the running watch

**Location:** `docs/overview.md:404`

**Issue:** "`Run-Shed`'s still-running self-bounce appends one entry per poll under a 1440-bounce budget" is no longer the whole story.
The awaiting and driver-exit waits append one entry per poll too, outside the budget, which is why card 10 reworded `maxStatusHistoryEntries`'s doc comment.
The overview sentence explaining the same cap was left stale.

**Fix:** Say the self-bounce appends one entry per poll while the child runs, awaits approval or waits for its driver, so the history outgrows the 1440-bounce running budget.

### [LOW] The records end-to-end test does not assert the run ends done

**Location:** `internal/battencli/records_integration_test.go:143`

**Issue:** Card 16's assertions include "the run ends `done` past `Worktree-Teardown`".
The test steps `Worktree-Teardown` and discards its result, then checks only that the pair is gone and the archive tag carries the records.
A teardown that removed the pair but still halted, for example on a failed remote deletion, would pass.

**Fix:** Assert the teardown step's `Outcome` is `Done` and its `State` is `done`.

### [LOW] New and changed Go comments are fixed-column wrapped mid-sentence

**Location:** across the diff; representative instances are `internal/battenshed/innerrun.go:136-158` (`Call`'s disposition table), `internal/battenshed/innerrun.go:259,302` (`callAwaiting`/`callDone` helpers), `internal/fabricengine/remove.go:26-28,64,78-82,143-144,282` (`ErrPairSiblingDirty`, `Remove`, `nameStrandedPortalTeardown`), `internal/fabricengine/cleanup.go:94-95,112-113,127-128,388-398,423` (new field and step comments), `internal/fabricengine/archive.go` (file header, `ErrArchiveFailed`, `archiveWeftTip`, `resolveArchiveTip`), `internal/loomcli/driverprompt.go` (`driverTeardownCommand`), `internal/battencli/wire.go:40-42,164-167,194-200`, `internal/battenshed/deps.go`, `internal/shedrecipe/entries_batten.go`, `internal/shedrun/paths.go`, `internal/loomengine/config.go:215`, `internal/loomcli/commitrecords.go`, and the new test files' headers and helper comments.

**Issue:** CONSTRAINTS.md's Comment Line-Break Convention requires one sentence per line in new and changed Go comments, with no column limit.
These comments break mid-sentence at a fixed column, for example `ErrArchiveFailed is the sentinel every archiveWeftTip error wraps, so a teardown caller can tell` / `a failed archive …`.
Some files in the same diff already comply (`internal/loomengine/driver.go`, `internal/reedengine/lifecycle.go`, `internal/loomcli/wiring.go`), so the convention is followed in some places and not in others.
This is not a line-width finding: the defect is the mid-sentence break.

**Fix:** Rewrap each new or changed comment paragraph to one sentence per line, breaking further at a semicolon between independent clauses.
Leave untouched legacy lines alone.

### [NIT] Recipe header comment narrates the change

**Location:** `contracts/recipes/batten-recipe.yaml:13`

**Issue:** "Row names and max_bounces are unchanged." describes this diff, not the recipe, and reads as noise once merged.

**Fix:** Drop the sentence.
