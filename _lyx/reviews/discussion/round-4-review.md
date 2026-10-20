---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:88 (D4 rewritten), :222 (D9 Bound)
    summary: The rewritten outcome names `lyx fabric pull` as its way forward, but Fabric.Pull refuses a branch the reconcile's own fetch already moved, so the way forward fails at every moving call site.
    fix: "In D4's rewritten bullet and D9, decide how the way forward succeeds. Either (a) change D9's Bound so Fabric.Pull's warp arm is no longer unchanged: its unpushed-commit test (now `@{u}..HEAD`, read before Pull's own fetch) treats a local commit reachable from a value of the tracking ref's reflog, read as D4's rewound test reads it, as pushed, so a rewritten branch whose tracking ref the reconcile already fetched takes Pull's existing anchor walk and reset instead of ErrWarpDivergedUnpushed; or (b) name a different way forward for rewritten that does not run through that test. Add an Acceptance criterion that runs `lyx fabric pull` on a rewritten pair after a moving reconcile and ends with the branch recovered, not only one that names the command."
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:110 (D4 Bound), :161-176 (D6)
    summary: The stated bounds list the teardown lock, the weft write lock and the loop kill waits, but the local teardown still waits on the hub workspace lock, runs a foreground gc, and the snapshot step's locks are unstated; the per-pair bound is not a bound for a hub of pairs.
    fix: "In D6, extend the list of the local mode's waits so each one is either a try-acquire or a named bound: (1) Remove's `regenerateAfterVerb` waits up to `hubWorkspaceLockWait` on the hub workspace lock; keep that wait and name it, or skip the regeneration in the local mode; (2) Remove's `housekeepStores` runs a foreground `git gc --auto` with no time bound; state that the local mode skips it, or that it runs and is bounded by nothing, and say which; (3) say whether the snapshot commit of the task worktree and of the weft worktree outside the record pathspec takes the weft write lock, and if so by try-acquire; (4) state that the loop-kill waits of `runLockWait` apply per pair and that a reconcile tearing down several ended pairs is bounded by their sum, or tear down at most one pair per reconcile wait. Then make D4's Bound sentence list the same waits, so the two places agree."
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:195-197 (D8 wake), :279 (Auto-mode assumptions)
    summary: Wake is the call site where an unreachable origin is the expected state, yet the record decides only that every site carries on, so the daemon lifts the hub pause and the paused drivers resume on the stale view the task exists to prevent.
    fix: "In D8's wake bullet, state what the wake reconcile does when either ls-remote fails. Recommended: retry both ls-remote calls inside one fixed bound (name its value), and when still unreachable keep the daemon's lift as today but name `unreachable` in the notice with the reconcile `lyx orch start` or `lyx cleanup` runs once the network is back; or keep the owned hub entry live like a D3 stop. Either way add the bound of what a driver can do in that window to the Auto-mode assumption, and add an Acceptance criterion for wake with an unreachable origin."
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:299-300 (Acceptance criteria)
    summary: One Acceptance bullet pins rewound and rewritten together, and none pins the boundary that D4 says separates them or the precedence of rewound over ahead on the same branch.
    fix: "Split the bullet at line 299 into two: a task branch rewound on origin by another machine, with and without an unpushed local commit, is reported rewound; a warp upstream with no unpushed commit rewritten to a head that is no ancestor of any earlier tracking-ref value is reported rewritten. Add one case to the rewound bullet with the same branch before and after the tracking ref was fetched to origin's head, so the precedence over ahead is pinned, and one case where origin's head is an ancestor of an earlier value but the local tip holds none of that value's commits, which D4 leaves to diverged."
---

### [BLOCKING] The rewritten way forward fails after the reconcile's own fetch

**Location:** `_lyx/discussion/decision-record.md:88` (D4 rewritten), `:222` (D9 Bound); code `internal/fabricengine/pull.go` (`Fabric.Pull`), `internal/gitrepo/push.go:253` (`HasUnpushed`).
**Issue:** D4 sends a rewritten branch to `lyx fabric pull` in the pair's worktree, "whose existing anchor walk and reset recover it".
`Fabric.Pull` reads `hadUnpushed` from `rev-list --count @{u}..HEAD` before its own `Fetch`, and returns `ErrWarpDivergedUnpushed` when it is true and the upstream was rewritten.
The reconcile fetches every moved head in D1 step 1, so by the time it judges a branch rewritten, the tracking ref is already at origin's rewritten head, so the local commits of the earlier value fall inside `@{u}..HEAD` and `HasUnpushed` reads true.
Pull then refuses with the double-conflict error and changes nothing, so the named way forward fails at orch start, wake, resume, Merge-In, Finalize and `lyx cleanup --apply`; only the dry run, which fetches into no ref, leaves Pull's own fetch as the first one.
D9's Bound says Pull's "warp arm is unchanged", so the record leaves no place where this is fixed, and its own rewound test (the reflog read that survives a fetch outside the reconcile) shows the author knew a fetch erases the plain tracking-ref evidence.
PATTERN-refusal-way-forward requires that a stop name a way forward a test reaches; the Acceptance criterion at line 299 only checks that the command is named.

### [MEDIUM] Waits the local teardown still makes are not all bounded in the record

**Location:** `_lyx/discussion/decision-record.md:110` (D4 Bound), `:161-176` (D6); code `internal/fabricengine/remove.go` (`regenerateAfterVerb`, `housekeepStores` call), `internal/fabricengine/hubworkspace.go` (`hubWorkspaceLockWait`), `internal/pairteardown/teardown.go` (`runLockWait`).
**Issue:** D6 correctly turns the teardown lock and the weft write lock into try-acquires and keeps `EndSession`'s two `runLockWait` waits, and D4's Bound says the reconcile waits "only for its own teardown's loop kill".
The local mode skips only the origin steps and the board write, so `Remove` still runs `regenerateAfterVerb`, which waits up to `hubWorkspaceLockWait` on the hub workspace lock, and `housekeepStores`, a foreground `git gc --auto` that has no time bound and runs at orch start and wake.
Neither is listed, so D4's "waits only for" is false as written.
The snapshot step of the one case commits paths in the task worktree and in the weft worktree outside the record pathspec before the try-acquires, and the record does not say whether it takes the weft write lock or how.
Each bound is per pair; at hub scope the reconcile tears down every ended pair in turn, so the total is the sum, which the record does not state.

### [MEDIUM] Wake with an unreachable origin lifts the pause on a stale view

**Location:** `_lyx/discussion/decision-record.md:195-197` (D8 wake), `:279` (Auto-mode assumptions).
**Issue:** The record settles the network gate at wake (an owned hub entry is passed for the reconcile's two calls and fetches, any other hub entry skips the reconcile and the notice names it).
It does not settle the first thing a woken machine usually lacks, a network.
The daemon's wake is detected from the clock gap, with no wait for connectivity, so the first `ls-remote` often fails, the outcome is `unreachable`, every call site carries on, and the daemon lifts its pause, which resumes the paused drivers on the very view the Goal says no machine acts on.
The Auto-mode assumption bounds the choice by "refusing would block all offline work" and "D10 still reports the later rejected push", which holds for orch start and cleanup but not for the one site whose purpose is to catch up before resuming.
This is a guard downgraded at a specific site with no bound stated for that site, which the attack-surface question asks the Decision to state.

### [LOW] Rewound and rewritten share one Acceptance bullet and their boundary is untested

**Location:** `_lyx/discussion/decision-record.md:299-300`.
**Issue:** D4 states the two outcomes are disjoint, separated by whether origin's head is an ancestor of an earlier tracking-ref value, and that rewound takes precedence over ahead.
The record is explicit and correct on this, so the finding is only about coverage: line 299 joins both outcomes in one bullet with a semicolon, line 300 covers a fetch outside the reconcile but not the precedence over ahead on one branch, and the case D4 leaves to diverged (origin's head an ancestor of an earlier value, local tip holding none of that value's commits) has no criterion.

## Rejected

- **Bolt entry merge and locks.**
  D5 names a boardengine entry point beside `boardCriticalSection` that takes neither `board.lock` nor `.gitrepo-push.lock`, loads a Store from origin's `board.json` and applies `validateWrite` and `checkRunLocks`, states the unit of a refusal as one entry or design-file change, and refuses a later change that depends on a refused one.
  `checkRunLocks(loaded, after)` is a per-write net comparison, so applying it per change against the board as the kept changes left it matches the code.
  Not a defect.
- **Ended pair with a busy driver and a dirty task worktree.**
  D6 decides it: the case passes force, which `RemoveRefusal` answers for dirtiness alone, a merge in progress still refuses, the snapshot tag keeps the uncommitted paths, and the snapshot tag marks a re-run.
  The busy notice's `lyx cleanup --apply` therefore succeeds, except where the pair is mid-merge, which refuses with `ErrMergeInProgress` and its own message.
  Not a defect.
- **Evidence of removal from origin.**
  D3 step 2 reads origin's Bolt tip and that tip's own history only, so a pull-first fast-forward past the removal does not change the answer; D5 and D10 restate the same evidence, and line 306 is the Acceptance criterion.
  Not a defect.
