---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: README.md:39
    origin: lens:code-affected-set
    summary: "README.md names the one-repository layout in two table cells only, so the record's D2 review criterion (clone description, `--single-repo` hazard, switch/reset/stash hazards) is unmet there"
    fix: "In README.md add a short subsection (after the tapestry table, or under the getting-started pointer) titled for a hub of one repository. It must carry the two clone forms (`lym fabric clone --shortname <short> --single-repo <project-url>` and `lym fabric clone <project-url>` over an origin holding `lym-bolt`), what the layout puts on code branches (`.lym-anchor` at the root, records under `<anchor>/_lym`, stripped at every fabric merge), the hazard that `--single-repo` cannot recognise the code repository of an existing two-repository hub and starts a separate hub over the same code, and the hazards that switching, resetting or stashing in the prime holds or drops its live records (a prime off `lym.primeBranch` refuses records commits; `git reset` or `git stash` can drop uncommitted records). Mirror the wording already in docs/getting-started.md."
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/fabricengine/add.go:694
    origin: lens:code-correctness
    summary: "One-repository Add with `--anchor` only stages the new marker, so the fork point's tracked records stay under the old anchor and the new anchor starts without them; the plan card names `moveForkedWeftToAnchor` for this step and the two-repository path moves the content"
    fix: "In the one-repository `anchorTo` branch of Topology.Add, move the forked worktree's tracked `_lym` content from the fork's anchor to anchorTo (the move `moveForkedWeftToAnchor` makes in the two-repository layout, adapted to the one worktree) in addition to `stageNewMarker`, and include both the moved paths and the marker in the first records commit with the Anchor-Change trailer. Add a row to `TestAdd_OneRepository` (`AddedAtAnotherAnchorCarriesTheTrailer`) where the parent tracks a file under its `_lym` (for example `_lym/config/probe.yaml`) and assert it sits under the new anchor in the new worktree and is gone from the old one."
  - id: F3
    severity: MEDIUM
    class: consistency
    location: pattern/PATTERN-run-fsm.md:39
    origin: lens:code-correctness
    summary: "The run-fsm background says a protected prime push stays owed and that the refusal names `lym fabric push`, but D7 and `IsPushOwed` treat protected as not owed and the refusal now names the record's own way forward"
    fix: "In pattern/PATTERN-run-fsm.md replace 'and the push stays owed' (line 39) with the statement that a `protected` push is not owed, so neither `FailedPushesOwedBy` nor `IsPushOwed` reads it, and the next records push of the prime carries the records once direct pushes are allowed. Change line 35 to say the act rule's refusal names the owed record's own way forward, `lym fabric push` in the owed worktree for a transport failure and `git push` in the prime for `operator_unpushed`."
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/battencli/commitstatus.go:233
    origin: lens:code-tests
    summary: "battencli's one-repository seam wiring (OneRepository, MergeOperation, Stalled, Protected, and the MergeActive swap) has no test; only loomcli has a one-repository seam row"
    fix: "Add a tier-0 test in internal/battencli that builds `newCommitStatusSeam` with stub deps where OneRepository is true and MergeOperation names an operation for one probe then clears, and assert Commit runs only after the wait, MergeActive is never called, and a Protected push failure returns nil. Add the two-repository counterpart row asserting MergeActive true still skips."
  - id: F5
    severity: MEDIUM
    class: design
    location: internal/fabricengine/merge_onerepository_integration_test.go:512
    origin: lens:code-tests
    summary: "The D5 rule that a merge verb over a parked merge refuses as already in progress before step 1 commits records is pinned for MergeIn only; Merge with and without Squash has no row"
    fix: "In TestMerge_OneRepository add rows that park a merge (conflict on code.txt through MergeIn in the task worktree, resolve and stage it, then run `Merge` and `Merge` with Squash in the receiving worktree) and assert a MergeGuardError with AlreadyInProgress, the resolution kept on disk and in the index, and no records commit made (HEAD unchanged)."
  - id: F6
    severity: MEDIUM
    class: design
    location: internal/fabricengine/replay.go:479
    origin: lens:code-tests
    summary: "D7 says a Warp-SHA trailer that names a commit not being replayed stays, but every replay row uses a trailer naming a replayed or fork-point commit, so rewriting every trailer would still pass"
    fix: "In TestReplay_OneRepository add a row where a local records-only commit carries a Warp-SHA trailer naming a commit that is not replayed (for example origin's own commit or the fork point itself) and assert that trailer is unchanged after ReplayWeftForTest while a sibling trailer naming a replayed commit is rewritten."
  - id: F7
    severity: MEDIUM
    class: design
    location: internal/fabricengine/originreconcilemoves.go:280
    origin: lens:code-tests
    summary: "The origin-reconcile case that maps a one-repository localCodeCommitsStop to BranchDiverged with the commits named has no test, though D7 lists origin reconcile as a replay site"
    fix: "Add a one-repository integration row for reconcileRecordsBranch (through the origin reconcile entry the existing two-repository rows use) with a local code commit and a moved origin, and assert the branch state is BranchDiverged, Commits names the local code commit, nothing moved, and a records-only-only divergence is replayed instead."
  - id: F8
    severity: LOW
    class: design
    location: internal/fabricengine/integrateweft.go:62
    origin: lens:code-tests
    summary: "integrateWeft's one-repository stop on a local code commit, and its records-only carry with paths moved, have no row; TestAnchorChange_OneRepository runs a clean move only"
    fix: "Add a TestAnchorChange_OneRepository row where the pair holds a local code commit and origin moved, and assert the integration returns a stop naming that commit with the branch unmoved."
  - id: F9
    severity: LOW
    class: design
    location: internal/fabricengine/remoteteardown.go:179
    origin: lens:code-tests
    summary: "The one-repository teardown error branches have no row, the lease-rejected kept outcome and the gate refusal of deleteOneRepositoryBranchOnRemote, and the viewRefError failure with its way forward"
    fix: "Extend the OneRepositoryTagThenBranchThenViewRef row of TestRunTeardownChild_RunsTheRemoteStepsInOrderUnderTheLock, or add a sibling row, where origin's branch moves between the archive read and the delete (assert RemoteKeptReason) and where the view ref delete fails (assert the failed-step way forward names the `git push origin --delete lym-view/<slug>` step)."
  - id: F10
    severity: LOW
    class: design
    location: internal/fabricengine/recordscommit.go:35
    origin: lens:code-tests
    summary: "The D4 acceptance that a held index.lock released within the bound lets the commit land has only a helper test; dropping the retry wrapper in the commit paths fails no test"
    fix: "Add a row to TestRecordsCommit_OneRepository that holds `<gitdir>/index.lock`, starts CommitAnchoredPaths, releases the lock after a short wait the test controls through a channel, and asserts the commit lands."
  - id: F11
    severity: LOW
    class: design
    location: internal/fabricengine/recordsduringmerge.go:237
    origin: lens:code-correctness
    summary: "If newPaired or saveMergeState fails after update-ref has moved HEAD, commitRecordsAroundMergeWith returns committed true with an error while the merge record's WarpStart still names the old HEAD, so a later MergeAbort resets to the old start and drops the records commit"
    fix: "Update the merge record (WarpStart, WeftStart) before moving HEAD, or on a record-save failure move HEAD back with `update-ref` to the old head so the commit is not half applied; return the error without a committed commit in that case."
  - id: F12
    severity: LOW
    class: consistency
    location: internal/runfsm/act.go:257
    origin: lens:code-affected-set
    summary: "Judge clears PushOwed, PushOwedError and PushOwedWorktree for a set-aside push but not the new PushOwedWayForward, so the sibling owed fields are no longer cleared together"
    fix: "In the `if aside.PushOwed` block of Judge add `in.PushOwedWayForward = \"\"` beside the other resets."
  - id: F13
    severity: LOW
    class: consistency
    location: internal/battencli/overview_test.go:716
    origin: handler
    summary: "TestLoadOverviewThresholds, which exists at the review base, now matches one thresholds message instead of any WARN, so an unexpected warning no longer fails its no-warning rows, and no Loosened-Check trailer names a measured cause"
    fix: "Keep the original any-WARN assertion and remove the spurious layout-read warning at its source by giving the test's prime a git repository (hermetic init) so `oneRepositoryAt` logs nothing, or add a Loosened-Check trailer to the commit that names the measured cause."
---

### [BLOCKING] README.md carries no D2 clone description or hazards

**Location:** README.md:39
**Issue:** The record's review acceptance criterion for D2 requires `docs/getting-started.md` and `README.md` to describe cloning a one-repository hub, what it puts on code branches, the `--single-repo` hazard on a two-repository hub's code repository, and the switch, reset and stash hazards in the prime. `docs/getting-started.md` carries all of it. The README diff changes two table cells (the Start row and the records-repository glossary row) and has no clone command, no `--single-repo` hazard and no prime hazard. Plan card 15 narrows the README to those cells, but the record is the measuring stick and the diff does not make the change it asks for.

### [MEDIUM] One-repository Add with `--anchor` does not move the forked records

**Location:** internal/fabricengine/add.go:694
**Issue:** In the one-repository branch Add calls `stageNewMarker` only. The tracked records at the fork point's anchor stay there and the new anchor holds just the origin record, while card 6 names `moveForkedWeftToAnchor` for this case and D5 defines an anchor change as moving `_lym` with the marker. The tests cover only a fork whose `_lym` holds no other tracked file.

### [MEDIUM] PATTERN-run-fsm background contradicts D7 on the protected class

**Location:** pattern/PATTERN-run-fsm.md:39
**Issue:** The background says a prime push failing as `protected` leaves the push owed. D7 and the code (`FailedPushesOwedBy`, `IsPushOwed`) read `protected` as not owed. Line 35 still says the refusal names `lym fabric push`, but `PushOwedWayForward` now names the record's own way forward.

### [MEDIUM] battencli's one-repository seam wiring is untested

**Location:** internal/battencli/commitstatus.go:233
**Issue:** `battenCommitStatusDeps` and `newCommitStatusSeam` gained the one-repository branch swap and three new deps. Only loomcli's `TestCommitStatusSeam_Real` has one-repository rows, so a break in battencli's swap fails no test.

### [MEDIUM] Merge over a parked merge is pinned for MergeIn only

**Location:** internal/fabricengine/merge_onerepository_integration_test.go:512
**Issue:** `Merge` reaches `commitReceivingRecords` through `syncSideBeforeMerge` and `prepareOneRepositoryMerge`, a separate path from `mergeIn`. Its guard order (already-in-progress before any records commit) is read correct in `merge.go`, but only the `MergeOverAParkedMergeRefusesAndKeepsItsResolution` row, which runs `MergeIn`, pins it.

### [MEDIUM] A non-replayed Warp-SHA staying put has no row

**Location:** internal/fabricengine/replay.go:479
**Issue:** `rewordWarpSHA` rewrites only trailers naming a replayed commit. All replay rows use trailers that name replayed commits or the fork point, so a rewrite of every trailer would still pass.

### [MEDIUM] Origin reconcile's one-repository code stop is untested

**Location:** internal/fabricengine/originreconcilemoves.go:280
**Issue:** The new `errors.As(err, &codeStop)` case maps to `BranchDiverged` and names the local commits. No one-repository origin-reconcile row reaches it.

### [LOW] integrateWeft one-repository stop has no row

**Location:** internal/fabricengine/integrateweft.go:62
**Issue:** The stop on a local code commit and the records-only carry are not exercised by the clean-move anchor-change test.

### [LOW] One-repository teardown error branches are untested

**Location:** internal/fabricengine/remoteteardown.go:179
**Issue:** The lease-rejected kept outcome, the gate refusal, and the `viewRefError` failure with its way forward have no row.

### [LOW] The index.lock retry has no real-path row

**Location:** internal/fabricengine/recordscommit.go:35
**Issue:** `TestRetryHeldIndexLock` pins the helper. No test holds a real `index.lock` and shows a records commit landing, so removing the wrapper at the commit sites fails nothing.

### [LOW] A failed merge-record update leaves WarpStart stale after HEAD moved

**Location:** internal/fabricengine/recordsduringmerge.go:237
**Issue:** After `update-ref` moves HEAD, a failure of `newPaired` or `saveMergeState` returns with the record's `WarpStart` still at the old HEAD. A later abort resets to the old start and drops the records commit, contrary to D4.

### [LOW] Judge does not clear the new PushOwedWayForward with its siblings

**Location:** internal/runfsm/act.go:257
**Issue:** The `aside.PushOwed` reset clears three owed fields and leaves `PushOwedWayForward`. Harmless today, since `Refusal` reads it only under the push-owed reason, but the fields are no longer cleared together.

### [LOW] TestLoadOverviewThresholds assertion narrowed without a trailer

**Location:** internal/battencli/overview_test.go:716
**Issue:** The test matched any `WARN` and now matches only the thresholds message, because the layout read logs its own warning for a prime that is no git repository. A different unexpected warning in a no-warning row now passes, and no `Loosened-Check:` trailer names a measured cause.

## Rejected

- Fork code-scope, Fabric-Merge trailer applied in both layouts: card 9's ImpactSummary states every fabric merge commit gains the trailer, so the plan licenses it.
- Fork code-scope, anchor-change trailer added in the two-repository move: card 8 states `moveWeftContent` adds the trailer in every layout.
- Fork code-scope, `MergeStateOperation` reporting cherry-pick, revert and rebase in the two-repository branch: card 10 defines `MergeStateOperation` without a layout split.
- Fork code-scope, D3 hub workspace file lists no sibling folders has no change or test: no sibling exists in a one-repository hub for `hubWorkspaceFolders` to list, so the decision holds with no code change; no evidence of a defect.
- Fork code-affected-set, reader-side `RecordsBranchName` calls in `reportcli`, `battencli` and `reportengine`: the plan's shared decision `outside-readers-unchanged` keeps them as they are, so the gap, if any, is in the answer key.
- Fork code-correctness, `rejectionRetryKind` ends retries for a protected refusal on any branch: the test row `ProtectedRefusalOfATaskBranchKeepsTodaysClass` shows such a refusal is classified `other`, so it never reaches `settleRejection`; no behavior change shown.
- Fork code-correctness, `lymcwd.OneRepository` and `oneRepositoryAt` disagree for a prime off its branch: D1 has `lymcwd` read the working tree and D7 stalls an off-branch prime, so the difference is the stated design.
- Fork code-tests, D1 missing-weft-sibling case has no row: the `AbsentAll_IsFallback` and sibling-only rows of `TestReadRecordedAnchor_MarkerSemantics` read a worktree with no root marker as two-repository.
- Fork code-tests, `statusPushRequest` one-repository branch has no row: `coalescePushSides` also treats an equal warp and weft path as one branch, so the pushed result is the same either way.
- Fork code-tests, `gitrepo` `MergeStartNoFastForward` and `remoteLines` have no unit test: `TestMerge_OneRepository` (fast-forward source) and the clone refusal rows reach both, per the reach report.
- Fork code-tests, merge trailer on a plain two-repository `--continue` has no row: `mergeCommitMessage` is one shared path, reached by the one-repository merge rows and the landing's commit message assertion.
- Declared PATTERN changes: all three entries land. The PATTERN-run-fsm and PATTERN-cwd-resolution overview lines are the compact form of the declared text and carry both clauses of each change.
- Redundancy report: every test it lists is unique, kept or not judged, so no test-economy finding applies.
