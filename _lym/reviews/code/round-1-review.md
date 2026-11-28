---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: "lens:code-correctness"
    location: "internal/fabricengine/clone.go#hubWorkOriginLacks"
    summary: "The --reset refusal counts a commit held only by a local tag as held by origin, and passes a hub holding a pending-teardown record, so --reset can still delete the only copy of an unfinished teardown's tip."
    fix: "In hubWorkOriginLacks, drop `--tags` from the `rev-list --count --branches --not --remotes --tags` exclusion so a commit that only a local tag holds counts as work origin lacks. Also name each record `PendingTeardowns(hubPath)` lists as work origin lacks, naming its slug and the re-run. Add rows to ResetOfAHubHoldingWorkOriginLacksIsRefusedAndKeepsTheHub for a local-only archive tag on a commit no branch holds, and for a pending-teardown record, each asserting the refusal and the kept hub."
  - id: F2
    severity: MEDIUM
    class: design
    origin: "lens:code-correctness"
    location: "internal/fabricengine/clone.go#existingHubError"
    summary: "existingHubError reads any hub whose Bolt checkout holds .git as whole. A clone killed after ensureBoardWorktree but before its anchor and layout steps now gets 'work in it' as its way forward and no longer the --reset clone that replaces it."
    fix: "In existingHubError, decide the advice by hubWorkOriginLacks(hubPath) instead of the Bolt .git test. Name the --reset clone (cloneResetCommand) when it finds nothing. When it finds work, name that work and the move aside. Add a row to TestCloneHub_HubExistsCheckPrecedesProbeInTwoArgForm, or a sibling test, for a hub with a Bolt checkout and no work, asserting the --reset clone is named."
  - id: F3
    severity: MEDIUM
    class: design
    origin: "lens:code-correctness"
    location: "internal/fabricengine/clone.go#hubWorkOriginLacks"
    summary: "hubWorkOriginLacks misses uncommitted edits in the prime code worktree, its stash and commits on a detached HEAD, so --reset still removes them. The change description's design decision exempts only the records store."
    fix: "In hubWorkOriginLacks, also name the prime code store's uncommitted or untracked changes (`git status --porcelain` non-empty), a `refs/stash` entry, and a detached HEAD whose commit `rev-list --count HEAD --not --remotes` counts above zero. Add a row to ResetOfAHubHoldingWorkOriginLacksIsRefusedAndKeepsTheHub for an uncommitted edit in the prime worktree."
  - id: F4
    severity: MEDIUM
    class: consistency
    origin: "lens:code-affected-set"
    location: "internal/fabricengine/doc.go:246"
    summary: "The new line 'Each command a record names as its re-run calls it' is false. Only `lym fabric remove --remote` and the cancel call SettleGonePairTeardown, while a cleanup record names `lym fabric cleanup --apply --remote` and a discard record names its caller's command."
    fix: "Replace the sentence with one naming the two callers: `lym fabric remove <slug> --remote` and the cancel each call it, so either finishes the teardown of a gone pair whose record names it."
  - id: F5
    severity: MEDIUM
    class: design
    origin: "lens:code-tests"
    location: "internal/fabricengine/cleanupremotewarp_integration_test.go#TestCleanupRemoteWarp_ApplyDeletesOnlyTheCandidate"
    summary: "Nothing tests that a failed tag push in the sweep keeps the branch and fills the entry's Error. Dropping the return after `entry.Error = err.Error()` in deleteLeftoverUnderLock fails no test."
    fix: "Add a subtest beside PushesTheLocalArchiveTagThatHoldsTheCandidatesWork. It refuses the tag push through SetArchiveTagPushHookForTest, then asserts the entry is not Deleted, its Error wraps the push failure, and origin's branch still sits at its tip."
  - id: F6
    severity: MEDIUM
    class: design
    origin: "lens:code-tests"
    location: "internal/battencli/catchup.go#fetchRunForCatchUp"
    summary: "No test fails on the base code for the catch-up's no-pair call site, which the board entry's fault names. Reverting catchup.go:237 to the raw records fetch fails nothing."
    fix: "Add a battencli test that calls fetchRunForCatchUp for a slug with no pair, over a one-repository hub whose prime records store lost its namespaced fetch refspecs, and asserts errors.Is(err, fabricengine.ErrRecordsLayout). If the per-process layout cache hides the lost refspecs, expose a cache reset through fabricengine's export_test.go seam, or build that hub in a fresh store the process has not read."
  - id: F7
    severity: MEDIUM
    class: consistency
    origin: handler
    location: "_lym/landing/summary.md"
    summary: "The Merge danger section calls the door two-way, although the diff now pushes archive tags to origin from the cleanup sweep and from `lym fabric remove --remote`. Its blast radius names fabric alone, though the diff also changes batten's catch-up fetch and gitrepo's leased-push error."
    fix: "In `## Merge danger`, call the door one-way, naming the archive-tag pushes to origin that the sweep and the gone-pair removal now make. Widen the blast radius to fabric, batten's catch-up and gitrepo's UpdateRemoteBranchLeased callers."
  - id: F8
    severity: LOW
    class: design
    origin: "lens:code-tests"
    location: "internal/fabriccli/remove_test.go#TestRemover"
    summary: "The `remote &&` guard before settleGonePair is unpinned. Every TestRemover row runs with remote true, so settling without --remote would fail no test."
    fix: "Add a wantRemote-false row to TestRemover. Its teardown returns the nothingLeft error, settled is true, wantCalls excludes 'settle', and the row asserts ErrPairNotFound is returned. Add a remote field to the table and pass it to r.run in place of the literal true."
  - id: F9
    severity: LOW
    class: design
    origin: "lens:code-tests"
    location: "internal/fabricengine/clone_singlerepo_integration_test.go#TestCloneSingleRepo_FailedSeedPushLeavesNothing"
    summary: "An authentication refusal on the board push, after the records push landed, is untested. That path withdraws the landed records branch through fail(..., recordsTip)."
    fix: "Add a row to RefusedCredentialRemovesTheHubAndNamesTheCredentials, or a sibling subtest, whose seed hook swaps in the refusing transport only for the board ref. Assert the credentials way forward, an empty container, and origin's records namespace empty."
  - id: F10
    severity: LOW
    class: consistency
    origin: "lens:code-tests"
    location: "internal/fabricengine/remoteteardown_test.go#TestRunTeardownChild_HoldsTheLockUntilItsInputCloses"
    summary: "The added PendingTeardowns assertion duplicates the NoRemoteStepsKeepsTheEarlierRecord row of TestRunTeardownChild_SettlesThePendingRecordOnceItEnds, which already pins that a child given no job keeps the earlier record."
    fix: "Remove the writePendingTeardown setup, the PendingTeardowns assertion and the added doc-comment line from TestRunTeardownChild_HoldsTheLockUntilItsInputCloses."
  - id: F11
    severity: LOW
    class: design
    origin: "lens:code-correctness"
    location: "internal/fabriccli/remove.go#settleGonePair"
    summary: "A settled gone pair returns an empty MutationRecord, though SettleGonePairTeardown pushed archive tags to origin on a Mutations it discards. The envelope then reports no mutation."
    fix: "Let SettleGonePairTeardown take the caller's *Mutations, or return its rec's snapshot beside settled. Then have settleGonePair put those mutations on the returned RemoveResult's MutationRecord, and update the cancel's caller to match."
  - id: F12
    severity: LOW
    class: design
    origin: "lens:code-correctness"
    location: "internal/fabriccli/remove.go#settleGonePair"
    summary: "When the removal's ErrPairNotFound names a stray path left at the task worktree's location, a successful settle replaces it, so the result never reports stray_path."
    fix: "Before settling, read the stray path the teardown reported (res.Removal.StrayPath from the teardown's result, or the path refuseWhenNothingRemains names) and carry it on the settled RemoveResult's StrayPath. Pass the teardown's result into settleGonePair to do so."
  - id: F13
    severity: LOW
    class: design
    origin: "lens:code-correctness"
    location: "internal/fabricengine/cleanupremotewarp.go#deleteLeftoverUnderLock"
    summary: "The landed check also counts any other local branch as holding the tip, and the sweep pushes only the slug's archive tags. A tip that only a local branch of another name holds is still deleted from origin."
    fix: "In deleteLeftoverUnderLock, after pushing the slug's archive tags, re-run the landed check with the local-branch glob dropped from checkUnlandedRemoteTip's exclusion list, so only remote-tracking refs and tags count. Keep the candidate with a Reason when that check fails. Add a subtest for a tip held only by another local branch."
  - id: F14
    severity: LOW
    class: design
    origin: "lens:code-tests"
    location: "internal/fabriccli/fabric.go#runRemoveWithFlag"
    summary: "The CLI wiring of settle is undriven. TestRemover stubs it, so nothing checks the re-run `lym fabric remove <slug> --remote` it passes."
    fix: "Add an integration row in fabriccli that runs `lym fabric remove <slug> --remote` over a hub holding a pending record of a gone pair. Assert steps is [pending_teardown] and the record is gone. With the slug's teardown lock held, assert the refusal names `lym fabric remove <slug> --remote`."
  - id: F15
    severity: LOW
    class: consistency
    origin: "lens:code-affected-set"
    location: "tools/sandbox/SANDBOX-FABRIC-SUITE.md:427"
    summary: "Sandbox scenario F12 still re-runs --reset against a real hub with no step for the new refusal of a hub holding a pair or an unpushed code commit, nor for the no---reset advice for a whole hub."
    fix: "In scenario F12, run the --reset over a hub holding no pair and no unpushed commit. Add a step that runs it over a hub with a pair and expects the refusal naming the pair worktree and the `mv` way forward, with the hub kept."
  - id: F16
    severity: LOW
    class: consistency
    origin: "lens:code-affected-set"
    location: "_lym/landing/summary.md"
    summary: "The summary says the --reset refusal comes before the clone touches the network. In the one-argument form, checkResetTarget runs only in resetHub, after probeWeftBinding has read origin."
    fix: "In the `--reset` bullet of `## What changes`, say the refusal comes before the clone touches the network in the two-argument and one-repository forms, and after the binding probe in the one-argument form."
  - id: F17
    severity: LOW
    class: design
    origin: handler
    location: "internal/fabricengine/clone.go#checkResetTarget"
    summary: "The printed way forward `mv <hub> <hub>.old` breaks the moved hub's linked worktrees, whose .git files and gitdir entries hold absolute paths. When <hub>.old already exists, mv moves the hub inside it instead."
    fix: "In checkResetTarget's and existingHubError's way forward, follow the move with the repair: `mv <hub> <hub>.old && git -C <hub>.old/<code-store> worktree repair` (and the same for the records store). Say to pick a name that does not exist yet. Keep the `.old` suffix in one named constant both messages use."
---

### [MEDIUM] --reset counts local-only tags as held by origin

**Location:** `internal/fabricengine/clone.go#hubWorkOriginLacks`
**Issue:** The commit count excludes everything `--tags` reaches, and a local tag is not on origin. This is the defect the cleanup-sweep fix of the same diff removes. A teardown child that ended before its tag push leaves an archive tag on this machine alone, together with a pending-teardown record that names it. That commit then counts as held, the record is not counted at all, and `--reset` deletes both.

### [MEDIUM] Partly cloned hub with a Bolt checkout loses its --reset way forward

**Location:** `internal/fabricengine/clone.go#existingHubError`
**Issue:** The whole-hub test is a `.git` at the Bolt checkout. A clone whose process dies after `ensureBoardWorktree` and before its excludes, `.lym`, anchor commit and layout resolve leaves a hub that passes that test. `RefuseHubWithoutBolt` also passes it. Its way forward is now "work in it", which nothing finishes.

### [MEDIUM] --reset misses uncommitted edits, stashes and detached commits in the prime code store

**Location:** `internal/fabricengine/clone.go#hubWorkOriginLacks`
**Issue:** Only linked worktrees and branch commits are read. The prime code worktree's uncommitted edits, `refs/stash` and a detached HEAD's commits are work origin lacks that `resetHub` still removes. The change description exempts only the records store.

### [MEDIUM] Package doc overstates who settles a gone pair's record

**Location:** `internal/fabricengine/doc.go:246`
**Issue:** The line says each command a record names as its re-run calls `SettleGonePairTeardown`. The cleanup sweep and a discard's caller other than the cancel do not call it.

### [MEDIUM] The sweep's failed tag push has no test

**Location:** `internal/fabricengine/cleanupremotewarp_integration_test.go#TestCleanupRemoteWarp_ApplyDeletesOnlyTheCandidate`
**Issue:** The keep-on-failure branch the doc promises is not driven. The seam to refuse the push already exists.

### [MEDIUM] The catch-up call site has no test that fails first

**Location:** `internal/battencli/catchup.go#fetchRunForCatchUp`
**Issue:** The board entry names `fetchRunForCatchUp` as the fault, and asks for a test that fails first. The added row drives `FetchRecords` over a pair, so reverting the call site fails nothing. The summary admits the call site is not driven.

### [MEDIUM] Merge danger is called two-way although the diff pushes outward

**Location:** `_lym/landing/summary.md`
**Issue:** The cleanup sweep and the gone-pair removal now push archive tags to origin, which the rubric's two-way bar excludes. The blast radius omits batten's catch-up and the callers of gitrepo's leased push.

### [LOW] The --remote guard on settling is unpinned

**Location:** `internal/fabriccli/remove_test.go#TestRemover`
**Issue:** Every row passes `remote` as true.

### [LOW] Board-push authentication refusal is untested

**Location:** `internal/fabricengine/clone_singlerepo_integration_test.go#TestCloneSingleRepo_FailedSeedPushLeavesNothing`
**Issue:** Only the records push is refused in the new row, so the rollback of a records branch that already landed is undriven.

### [LOW] Duplicated assertion on the kept record

**Location:** `internal/fabricengine/remoteteardown_test.go#TestRunTeardownChild_HoldsTheLockUntilItsInputCloses`
**Issue:** The `NoRemoteStepsKeepsTheEarlierRecord` row already pins the same behavior.

### [LOW] A settled removal reports no mutations

**Location:** `internal/fabriccli/remove.go#settleGonePair`
**Issue:** The archive-tag pushes are recorded on a `Mutations` that `SettleGonePairTeardown` discards.

### [LOW] Settling drops a stray path report

**Location:** `internal/fabriccli/remove.go#settleGonePair`
**Issue:** The not-found error that names a kept stray path is replaced by a success that carries no `StrayPath`.

### [LOW] The sweep's landed check still trusts other local branches

**Location:** `internal/fabricengine/cleanupremotewarp.go#deleteLeftoverUnderLock`
**Issue:** `checkUnlandedRemoteTip` excludes every local branch from the count, and no push covers a tip that another local branch holds.

### [LOW] The remove CLI's settle wiring is undriven

**Location:** `internal/fabriccli/fabric.go#runRemoveWithFlag`
**Issue:** The re-run string reaches the operator in a held-lock refusal, and no test reads it.

### [LOW] The sandbox fabric suite's reset scenario predates the refusal

**Location:** `tools/sandbox/SANDBOX-FABRIC-SUITE.md:427`
**Issue:** The scenario can now meet the new refusal on a hub that holds pairs, and it has no step for the new advice.

### [LOW] The summary's network-order claim is false for the one-argument form

**Location:** `_lym/landing/summary.md`
**Issue:** The one-argument clone checks the reset target only inside `resetHub`, after the binding probe.

### [LOW] The move-aside way forward breaks linked worktrees

**Location:** `internal/fabricengine/clone.go#checkResetTarget`
**Issue:** Moving a hub leaves every linked worktree's absolute gitdir links stale. A target that already exists takes the hub inside it.

## Rejected

- The code-tests lens's LOW on the unpinned `--tags` exclusion is subsumed by F1, whose fix removes that exclusion and adds the rows that pin the change.
- The code-scope lens raised no finding; every change traces to a goal point.
