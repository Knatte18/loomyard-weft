---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: lens:code-affected-set
    location: internal/fabricengine/clone_singlerepo_integration_test.go:596
    summary: New message expectations duplicate the owned namespace and board-branch constants.
    fix: Build the new expected fragments at lines 596 and 604–605 with fabricengine.RecordsNamespaceRoot and fabricengine.OneRepositoryBoardBranch instead of literal refs/lym/ and branch.main; keep the remaining expected wording literal.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/pushclassified.go:240
    summary: The printed upstream repair fails for paths with spaces and shell-significant branch names.
    fix: Use internal/shell quoting for the path, complete branch config keys and merge-ref values in both generated git config commands. Extend the existing recovery table with a spaced destination and a shell-significant records branch; execute the rendered commands and verify the resulting settings and successful push.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/destroy.go:1787
    summary: The board failure claims records restoration before the restore succeeds.
    fix: Say the seed attempts to return the records branch, or append successful-restoration wording only after restoreSeededRecordsBranch succeeds. Extend the existing killed-push test with successful board withdrawal followed by a failed records restore, asserting the surviving refs and accurate failure wording.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/clone_singlerepo_integration_test.go:579
    summary: The refusal test omits unreadable origin and a committed local board facing an unknown origin commit.
    fix: Add rows to BoardWithNoUpstreamIsNeverPushed for unreachable origin and a committed local board whose unrelated origin tip is absent locally. Assert retry or join guidance respectively, no reset guidance, no failed-push record and unchanged origin refs where readable. Keep the unborn-board row as its separate boundary case.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/clone_singlerepo_killedpush_integration_test.go:55
    summary: No test checks the new withdrawal wording when another writer owns the observed ref.
    fix: Extend the existing killed-push table with unknown-outcome records and board pushes whose observed origin ref is neither the pushed tip nor the leased tip. Assert that the other writer's ref survives and that the error does not claim this seed withdrew that ref.
---

### [BLOCKING] Use the owned-name constants in the new expectations

**Location:** internal/fabricengine/clone_singlerepo_integration_test.go:596

**Issue:** The new expectations spell `refs/lym/` and `branch.main` directly.
`PATTERN-code-conventions` requires one defining constant for these names, including owned records branches.
The constants already exist in `recordslayout.go` and are used elsewhere in this test.
The round instructions make every violation of a listed constraint blocking.

### [MEDIUM] Quote the recovery command arguments

**Location:** internal/fabricengine/pushclassified.go:240

**Issue:** The commands interpolate the path, config keys and merge refs without shell quoting.
A destination such as `/home/a/My Projects/hub/_bolt` makes `git -C` receive the wrong directory.
A records branch containing `$name` also changes when the shell expands the command.
The existing test rebuilds argument arrays, so it does not exercise the displayed commands' shell syntax.
This leaves the requested recovery unusable for supported inputs.

### [MEDIUM] Do not claim restoration before its result

**Location:** internal/fabricengine/destroy.go:1787

**Issue:** The new cause says the seed returns the records branch to its previous tip.
The subsequent `fail` closure attempts that restoration and can fail its lease after another writer moves the ref.
The final error then retains the affirmative statement and appends that restoration also failed.
This contradicts the task's requirement for accurate withdrawal wording.
The test lens's restoration-failure coverage finding is consolidated here.

### [MEDIUM] Cover the refusal's remote failure and ancestry decision

**Location:** internal/fabricengine/clone_singlerepo_integration_test.go:579

**Issue:** The another-board row uses `CloneHub` without committing its orphan board.
It exercises `ErrNoCommits`, bypassing the ancestry check against an origin commit absent locally.
The table also contains no unreachable-origin row for the new retry guidance.
These are distinct runtime behaviors, including the guarantee that refusal writes no failed-push record.

### [MEDIUM] Cover the false withdrawal result

**Location:** internal/fabricengine/clone_singlerepo_killedpush_integration_test.go:55

**Issue:** The existing rows observe either the seed's pushed tip or the original leased tip.
None reaches the `(false, nil)` result for a different writer's tip.
That result now determines whether the caller claims withdrawal, so the new negative wording needs coverage.

## Review evidence

The reviewed range is `f660ac101279ab085ffbc815ebb3d1422d894030..HEAD` at `a32d529bdf81329010990510b4fb65dda4e3e1cb`.
The board entry and decision record agree, and every diff purpose traces to that scope.
The root-cause trailer identifies the missing origin read that the change adds.
No weakened assertion, redundant new test or new wait was found.

An origin commit absent locally is deliberately treated as not ancestral, so the join branch is reachable.
The reset and plain-join forms agree with the clone's board-presence rules.
The namespace mapping and records config location used by the repair are correct apart from shell quoting.
The unreadable-origin refusal names retry; the unborn-board case names join.
The records-key read failure propagates context without guessing settings or recording a failed push.
The refusal-way-forward background binds its trailing-clause requirement to webster, shed and loom, not fabric.
No finding expands that requirement to this config-read error.

The withdrawal boolean preserves the existing control flow in every return branch.
The diff changes wording and a return value, not record formats, config meanings or outbound mutation behavior.
The landing summary's two-way merge-door assessment therefore holds for this diff.

The targeted tier1 run could not acquire a gate slot because the sandbox made the hub's gate directory read-only.
No test result is claimed from that attempt; this environment failure is not a finding against the diff.
