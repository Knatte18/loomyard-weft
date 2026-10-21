---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/gitrepo/gogit.go:88
    summary: reindexIfPacksChanged leaves go-git's pack index to be rebuilt lazily by concurrent readers holding only the shared lock, so one reader sees a half-built index and reports object not found
    fix: In reindexIfPacksChanged, right after storer.Reindex() and while r.goGitMu is still held exclusively, force the rebuild by calling storer.HasEncodedObject(plumbing.ZeroHash) and ignoring its result (it runs requireIndex, and plumbing is already imported). Add a sentence to the function's comment saying the rebuild is forced under the exclusive lock because go-git rebuilds the index unsynchronized. TestGoGit_StandaloneRepo/two_readers_over_a_stale_pack_index_share_one_reindex then needs no change.
  - id: F2
    severity: MEDIUM
    class: decision
    location: PATTERN.md:11
    summary: Five overview entries other than the two declared ones differ from main's text, four of them with the qualifier cut from the overview line
    fix: Restore main's overview line verbatim for PATTERN-durable-vs-ephemeral-state, PATTERN-hub-suffix, PATTERN-push-both-sides, PATTERN-edit-directive and PATTERN-recipe-setting-lookup (with no new background link or pattern/PATTERN-recipe-setting-lookup.md, which the diff adds). Restoring costs 267 bytes, so the overview becomes 16003 bytes against the 15743-byte base and the 15564-byte headroom line, and internal/pattern's Headroom reports it. Pay for it by shortening the overview lines of the two declared entries PATTERN-origin-reconcile and PATTERN-push-failure-visible, which carry 260 bytes together and still leave 16003 minus their savings above the base. If the overview cannot fit at or under 15743 bytes with main's five lines intact, stop and ask ly:orch whether to raise the declared-change list to name the rewritten entries, rather than shortening undeclared entries again.
  - id: F3
    severity: LOW
    class: consistency
    location: internal/reedcli/watchdog_integration_test.go:347
    summary: The shared loop's idle window equals watchdogTestWait, so it does not exceed the longest gap DownThenUpDoesNotKillDaemon can wait for the re-upped session
    fix: Set timing.IdleCycles to twice the wait, int(2 * watchdogTestWait / watchdogTestCycle), and say in the comment that the window spans the Up call plus the tests' own wait budget. Nothing in the loop's later subtests relies on an idle exit, so the larger value changes no other subtest.
---

### [BLOCKING] TestGoGit_StandaloneRepo/two_readers_over_a_stale_pack_index_share_one_reindex

**Location:** internal/gitrepo/gogit.go:88
**Issue:** The Publish verify log (`.lyx/verify/verify-11.log`) shows reader 0 failing with `object not found` after the retry.
The branch's own diff does not touch `gogit.go` or `gogit_test.go`, and the merge-in commit's diff does not touch `internal/gitrepo` either, so the defect is older than the branch.
The test passed alone and in 40 repeated runs under `--tags integration`; the log's run was a loaded full-module run.
The cause is in the code and in go-git's source.
`reindexIfPacksChanged` calls `storer.Reindex()`, which only sets the storer's index to nil (`storage/filesystem/object.go:74`).
The index is rebuilt by `requireIndex` on the next read (`object.go:53`), which assigns an empty map before it fills it and takes no lock.
`readGoGit` then runs the retry under `r.goGitMu.RLock()`, which is shared, so two readers can enter `requireIndex` together.
The second sees a non-nil index that is not yet filled and returns `ErrObjectNotFound`, exactly the failure in the log.
With two readers the test hits this whenever both reach the rebuild together, which a loaded run makes likely.
The package's one-reindex contract does not cover the rebuild itself.

### [MEDIUM] Undeclared PATTERN overview edits

**Location:** PATTERN.md:11
**Issue:** The directive declares changes to `PATTERN-origin-reconcile` and `PATTERN-push-failure-visible` only, and says every other entry is judged by the overview's current text.
`git diff main -- PATTERN.md` (main is the merge base) shows five more entries changed:
- `PATTERN-durable-vs-ephemeral-state` loses "nothing a run needs lives only under `.lyx`".
- `PATTERN-hub-suffix` loses "no code recognises retired `-HUB`".
- `PATTERN-push-both-sides` loses the per-transition detached-push wording.
- `PATTERN-edit-directive` loses "orch stencils are exempt".
- `PATTERN-recipe-setting-lookup` is cut to "looked up by key" and gains a new `[background]` link and file.

The four qualifiers survive in the existing background files, but the overview is what agents get inlined, so a reviewer or implementer reading only the overview now lacks the orch-stencil exemption and the other qualifiers.
The overview is 15736 bytes, below main's 15743, so the headroom check passes; the cuts exist to buy that room.
Restoring all five lines and still fitting needs a choice the diff cannot settle alone, which is why this is a `decision` finding.

### [LOW] Idle window of the shared watchdog loop has no margin

**Location:** internal/reedcli/watchdog_integration_test.go:347
**Issue:** `timing.IdleCycles = int(watchdogTestWait / watchdogTestCycle)` is 50 cycles of 200ms, 10s, the same as `watchdogTestWait`.
`DownThenUpDoesNotKillDaemon` runs `eng2.Down()` and `eng2.Up()` before it starts its 10s wait, and `eng1` is already down by then, so the hub is idle from the down on.
The window therefore has to cover the `Up` call plus a wait of up to 10s, which it cannot do with no margin.
The usual case is a short gap, and a cycle runs longer than 200ms under load, so this fails only on a slow `Up` followed by a slow wait.
No later subtest depends on the loop exiting idle: `ReEntryReReadsFlippedConfig` asserts it stays running and ends by cancelling the loop.

## Focus directive answers

- Third and fourth directive points: not a defect.
  `resetKeepOrTag` (`destroy.go:1806`) tags the starting tip with `setAsideTag` (a tag creation, which the destruction chokepoint does not govern) and returns an error naming the tag and the `git reset --keep` that restores it.
  `restoreReplayTip` wraps that error, and `restoreOnFailure` in `replayBolt` joins it into the returned error.
  In `cherryPickOnto` every return after `ResetKeep(onto)` goes through `resetBack`: `CommitDetail`, the `before` read, the pick conflict, the `after` read and the final `CurrentSHA`.
  The only earlier returns are the `ResolveSHA` check and the reset's own refusal, where the branch did not move.
  `replay_integration_test.go:661` covers the tagged refusal.
- Per-card mechanical checks, comment conventions and test economy: nothing further found in the range I read.
  The range is large and I read the focus targets and the gitrepo, gitexec and PATTERN diffs closely, and the rest only in the destroy and replay paths.
- The one `Pattern-Departure:` trailer I found names `PATTERN-fabric-vocabulary`, whose entry forbids the card's spelling in a non-owner file, so it holds.
