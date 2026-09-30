---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/plan/01-shedengine-budget-exempt.md:18
    summary: An exempt Stuck still enters the budget arm, so after exactly N counted Stucks the first awaiting or driver-exit bounce blocks with the budget exhausted, and card 10's one-short test cannot see it
  - id: F2
    severity: MEDIUM
    location: _lyx/plan/05-loom-status-commit-records.md:27
    summary: No card proves against real git that the records pathspec commits friction notes, an archive rename and the drive report onto the weft tip; card 16 stubs that link and cards 5 and 7 test only composition and stub closures
  - id: F3
    severity: MEDIUM
    location: _lyx/plan/09-batten-innerrun-seams.md:33
    summary: reedengine Status errors when the task session is absent (requireSessionLocked), so DriverAlive as specified returns an error that card 10 treats as live, delaying every gone-session done child by the full grace window
  - id: F4
    severity: MEDIUM
    location: _lyx/plan/10-batten-innerrun-awaiting-donewait.md:67
    summary: Several docs statements the change falsifies are left untouched - overview batten section (halted-child restart, 12-hour uncommitted watch, teardown refusal text, archive step), battenshed doc.go report path and never-liveness sentence, battencli maxStatusHistoryEntries comment
  - id: F5
    severity: LOW
    location: _lyx/plan/10-batten-innerrun-awaiting-donewait.md:42
    summary: The done-seen marker is never cleared, so a stale marker from an earlier run of the same slug makes the done arm skip the driver-exit wait; the past-grace warning is also never asserted
  - id: F6
    severity: LOW
    location: _lyx/plan/13-fabric-pairbranch-cleanup-archives.md:25
    summary: Cleanup gains ArchiveTag only, so a weft repo with no origin drops the archive skip reason the decision record requires in the result
  - id: F7
    severity: LOW
    location: _lyx/plan/14-fabric-sibling-dirty-sentinel.md:17
    summary: The sentinel test uses an unspecified uncommitted file, not the load-bearing case of an untracked report in a new drive-reports directory
  - id: F8
    severity: LOW
    location: _lyx/plan/01-shedengine-budget-exempt.md:31
    summary: ImpactSummary omits that Routing.Bounces (status progress) also reads episodeStuckCount, and the open risk's status-rendering check is not carried
  - id: F9
    severity: NIT
    location: _lyx/plan/10-batten-innerrun-awaiting-donewait.md:29
    summary: The decision record says the plan verifies that lyx loom start --no-attach on an awaiting run launches a driver; the plan never states it
---

### [BLOCKING] An exempt Stuck is still blocked by an exhausted budget

**Location:** `_lyx/plan/01-shedengine-budget-exempt.md:18`, with the matching test at `_lyx/plan/10-batten-innerrun-awaiting-donewait.md:61`.

**Issue:** `Shed.stepLocked`'s Stuck case (`internal/shedengine/run.go:266-285`) checks `episodeStuckCount(st.History, def.Name) >= effectiveMaxBounces(...)` for every Stuck outcome, before routing to `OnStuck`.
Card 1 only makes `episodeStuckCount` skip exempt entries.
A Run-Shed row whose running child used exactly N counted Stucks (all allowed: the Nth passes because the check then reads N-1) and then turns `awaiting` or `done` returns an exempt Stuck; the check reads N >= N and blocks the run with `ReasonBounceBudgetExhausted`.
The decision record's acceptance criterion says the driver-exit wait "neither consumes nor is blocked by the running budget", and its budget Decision says approval time is unbounded by design.
Card 1's test ("fewer than N counted ones") and card 10's test ("N-1 counted Run-Shed Stucks and a `MaxBounces` of N") both stop one short of the case that fails.

**Fix:** In card 1, the budget arm applies only to a counted Stuck: an exempt Stuck skips the `episodeStuckCount` comparison and routes to `OnStuck` whatever the counted total.
Add a card 1 test: a row with `MaxBounces` N and N counted Stucks already in the episode routes an exempt Stuck to `OnStuck`, then still blocks on the next counted Stuck.
In card 10, seed the history with N counted Run-Shed Stucks (the budget fully spent), so the test proves the done wait is not blocked.

### [MEDIUM] The real records pathspec is never exercised against git

**Location:** `_lyx/plan/05-loom-status-commit-records.md:27`; `_lyx/plan/07-loom-commit-records-verb.md:33`; `_lyx/plan/16-batten-records-integration.md:18`.

**Issue:** Focus item 1's question, settled: the link is covered only by unit tests.
Card 16's `Spawn` stub commits the friction note and drive report itself through `fabricengine.CommitAnchoredPaths`, standing in for `statusCommitPathspec`; card 5 tests only the pathspec slice's composition, and card 7 tests `commitRecordsVerb` over stub closures with no git.
Nothing proves that the real `loomCommitStatusDeps` commit carries `_lyx/loom/...` and `_lyx/shed/<slug>/drive-reports/...` onto the weft tip, that staging `_lyx/loom` whole also stages a friction archive rename's deletion (the Shared Decision's whole rationale), or that `commit-records` over a clean tree is a no-op success (an acceptance criterion).
`internal/loomcli/wiring_commitstatus_integration_test.go` already drives the same seam over a real hubforge pair for the reviews directory (`TestCommitStatusSeam_Real_CommitsTheRoundRecord`, `TestCommitStatusSeam_Real_ArchiveRenameCommitsAdditionAndDeletion`), so the pattern exists.
Card 16 cannot call loomcli's unexported seam from battencli, so the proof belongs in loomcli.

**Fix:** Card 5 also edits `wiring_commitstatus_integration_test.go`: a friction note and a drive report written before a transition land in the weft HEAD commit and leave `_lyx/loom` and the drive-reports directory clean; a committed friction directory renamed to a `friction-<ts>` sibling is recorded as both an addition and a deletion.
Card 7 adds `commitrecords_integration_test.go` over the same fixture, driving `commitRecordsVerb` with the real `loomCommitStatusDeps`: an uncommitted drive report is committed and pushed, and a second call over the now-clean tree succeeds with no new commit.
Card 16's intent names those tests as the proof of the link its stub stands in for.

### [MEDIUM] DriverAlive treats an absent reed session as an error

**Location:** `_lyx/plan/09-batten-innerrun-seams.md:33`.

**Issue:** `reedengine.(*Engine).Status` calls `requireSessionLocked` (`internal/reedengine/lifecycle.go:1001-1024`), which returns a plain `errors.New(noSessionMessage(...))` when the task worktree's tmux session is not up.
It never boots anything, so card 9's "confirm Status() does not boot" check passes, but the absent-session case comes back as an error.
Card 10 (and the `liveness-error-is-live` Shared Decision) treats a liveness error as a live strand, so every done child whose session is gone (a machine restart, a manual `lyx reed down`) waits the full 900 s grace, where the decision record says "When the strand is gone, it returns Done."
There is no exported sentinel or session probe to tell the case apart, and matching the message text would be fragile.
Card 9's tests also cover only non-nil-ness and laziness, never the closures' answers.

**Fix:** Card 9 adds an exported sentinel `reedengine.ErrNoSession`, wrapped with `%w` by `requireSessionLocked` (message unchanged), and `DriverAlive` maps `errors.Is(err, reedengine.ErrNoSession)` to not live; any other error stays an error.
Card 9 gains a reedengine test that `Status` on a worktree with no session satisfies `errors.Is(err, ErrNoSession)`, and `internal/reedengine/...` joins both `go test` lines of the verify section.

### [MEDIUM] Docs the change falsifies are left unedited

**Location:** `_lyx/plan/10-batten-innerrun-awaiting-donewait.md:67`; `_lyx/plan/15-batten-teardown-records-remedy.md:1`.

**Issue:** Focus item 7's question, settled: cards 7, 8, 10 and 12 reach the ly-drive skill, `CONSTRAINTS.md`, `internal/fabricengine/doc.go`, the awaiting sentence of `docs/overview.md`'s batten section and one paragraph of `internal/battenshed/doc.go`, but several statements stay false after the plan:
- `docs/overview.md`'s batten section: "batten cannot restart the child's driver" (now it does, for an approved awaiting child); "prime's own pair carries an uncommitted change … up to the full 12 hours" (an awaiting wait is unbounded); teardown's "fabric's own refusal as the `stuck_reason` — the operator cleans the child and re-steps" (card 15 replaces it for uncommitted run records with `lyx loom commit-records` and a resume); and the teardown paragraph never mentions the archive tag.
- `internal/battenshed/doc.go`: a stopped driver leaves "its report under the task worktree's .lyx/shed/<slug>/" (now `_lyx/shed/<slug>/drive-reports/`), and "the InnerRun row watches the child's persisted status file, never the driver's own liveness" (the done arm now reads liveness).
- `internal/battencli/arm.go`'s `maxStatusHistoryEntries` comment says the self-bounce appends "under a 1440-bounce budget"; exempt bounces now append beyond it.
The decision record's Documentation Lifecycle constraint requires module docs updated wherever they name the moved paths or the awaiting flow.

**Fix:** Card 10's docs list covers each of the overview and `battenshed/doc.go` statements above, and card 10 adds `internal/battencli#maxStatusHistoryEntries` to its targets for the comment.
Card 15 adds `docs/overview.md` as a `Prosa` target: the teardown sentence names batten's own remedy for uncommitted run records and the archive tag the removal pushes.

### [LOW] The done-seen marker has no end of life

**Location:** `_lyx/plan/10-batten-innerrun-awaiting-donewait.md:42`.

**Issue:** Focus item 5's question, settled: the acceptance arms (exempt Stuck while live, Done when gone, Done past grace, the counted-budget case) are all carried, and the marker survives a resumed batten run because it is "never overwritten while present".
Its end is unspecified: the marker lives in batten's prime-side scratch directory (`shedrun.ScratchDir(prime, slug)`), which nothing clears, so a later batten run for the same slug (slug reuse is what the archive tag exists to allow) finds an old timestamp and returns Done at once, killing a driver still writing its stop report.
The test list also never asserts the past-grace warning the acceptance criterion names.

**Fix:** The done arm removes the done-seen marker when it returns Done, and any non-done Call removes a stale one before its own arm runs; a failed removal other than "absent" is a hard error like the other marker failures.
Add tests: a marker present while the child is `running` is removed, and a Done return leaves no marker; the past-grace test asserts the warning through the package's log capture, or the card states why it cannot.

### [LOW] Cleanup drops the no-origin archive skip reason

**Location:** `_lyx/plan/13-fabric-pairbranch-cleanup-archives.md:25`.

**Issue:** Focus item 3's question, settled: Prune deletes no weft branch (`removeStalePair` removes the worktree only), so reading the record's "prune" as `Cleanup` is faithful, and Cleanup's `deleteWeftBranchOnRemote` runs only when `entry.Deleted` is true, so it is held behind the archive.
But card 13 gives `CleanupBranchEntry` only `ArchiveTag`; when the weft repo has no origin, `archiveWeftTip` returns a skip reason Cleanup has nowhere to put, where the record says "A weft repo with no origin skips the tag with a reason in the result".

**Fix:** `CleanupResult` gains a once-per-verb `ArchiveSkippedReason` (`json:"archive_skipped_reason,omitempty"`), set from the first skip, matching how `RemoteSkippedReason` is reported once per verb; card 13 targets `internal/fabricengine#CleanupResult` and its test covers the no-origin case.

### [LOW] The sentinel test does not pin the load-bearing case

**Location:** `_lyx/plan/14-fabric-sibling-dirty-sentinel.md:17`.

**Issue:** Focus item 4's question, settled: `refuseDirtyWeftWorktree` runs `worktreeDirty(scopeAll, …)`, which is plain `git status --porcelain` with untracked files shown, and the weft's `.git/info/exclude` excludes only `.weft/`, `.lyx/`, lock files and swaplocks, so an uncommitted drive report in a new directory does reach the refusal and `ErrPairSiblingDirty`.
Card 14's test says only "an uncommitted file in its `_lyx` tree", which a modified tracked file satisfies without proving the case the done-wait Decision rests on.

**Fix:** The test's dirty file is a new, untracked file inside a not-yet-tracked `_lyx/shed/<slug>/drive-reports/` directory.

### [LOW] Card 1's ImpactSummary and the history-growth risk

**Location:** `_lyx/plan/01-shedengine-budget-exempt.md:31`.

**Issue:** `episodeStuckCount` has two callers: the budget arm in `run.go` and `Routing.Bounces` in `progress.go`, which feeds the status envelope's `progress` bounce count.
The ImpactSummary names only "one skip in the budget counter".
The decision record's open risk also asks the plan to check `lyx batten status` rendering at a few thousand entries; the plan adds a write-cost benchmark only.

**Fix:** The ImpactSummary states both callers (budget arm and status progress count, both now counting only non-exempt Stucks).
Card 1 notes that batten's status envelope already bounds history through `maxStatusHistoryEntries` with the true `history_length`, so rendering stays bounded at any history length.

### [NIT] The start-on-awaiting assumption is not stated

**Location:** `_lyx/plan/10-batten-innerrun-awaiting-donewait.md:29`.

**Issue:** The record's auto-mode assumptions say `lyx loom start --no-attach` on an `awaiting` run launches a driver, "the plan verifies this against `start.go`".
The plan never mentions it.
The code bears it out: `startCmd`'s step 3 spawns the recorded driver unless one is alive, with no state gate.

**Fix:** Card 10's intent states the verified premise in one sentence.

## Verdicts on the remaining focus items

Focus item 2 (whole `_lyx/loom` pathspec): not a defect.
No loomengine accessor joins `lyxdirs.LyxDirName` with `loomDirName` today; every other `loomDirName` path sits under `.lyx/loom`, and card 3's new `LoomDurableDir` doc comment declares everything under it tracked run content, so the wider pathspec adds no content the record does not license.

Focus item 6 (cards 9 and 10 granularity): not a defect.
Card 9's seams are wiring from prime plus registry validation, testable through `wire_test.go` and `entries_batten_test.go` without card 10, and F3 gives it behavior tests of its own.
