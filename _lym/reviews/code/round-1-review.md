---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/fabricengine/destroy.go:1415
    summary: RemoveStrayTaskPath is a destructive executor that skips the ownership and dirtiness checks, and the PATTERN-fabric-git exception for removeWorktreeLeftover is neither met nor amended
    fix: PATTERN-fabric-git's Destruction chokepoint allows one exception, removeWorktreeLeftover, whose ownership and dirtiness come from a gated git worktree remove its caller just ran on the same target. RemoveStrayTaskPath runs no such git call. Either route the delete through the destructive gate with a declared ownership kind and a dirtiness declaration, or amend the Destruction chokepoint text in pattern/PATTERN-fabric-git.md to name RemoveStrayTaskPath as a second exception. The amendment states that the caller proves ownership, that the function refuses a registered linked worktree and a directory holding a .git entry, and that --force and dirtiness do not apply. Record it with a Pattern-Departure trailer on the fixing commit. Put the removeWorktreeLeftover doc comment back to the narrow wording it had before this branch, unless the exception text is widened to match it.
    origin: handler
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/battencli/wire.go:408
    summary: no test pins that teardownRequest sets ClearStray, so Worktree-Teardown could lose the stray delete with every test green
    fix: Add one assertion on teardownRequest's result, in the battencli test that already covers teardownRequest if one exists, or as a row in a table over teardownRequest. It asserts ClearStray is true, beside the Landed and Remote fields it already pins.
    origin: lens:code-tests
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/fabricengine/pairparts.go:25
    summary: PairParts.StrayPath and the Any() count are covered only through a fake PairParts value, so a broken inspectPair mapping passes every test
    fix: Add a row to the fabricengine PairParts test (or to the stray-directory integration test) that plants a plain directory at a slug's task worktree path and asserts PairParts(...).StrayPath is true, TaskWorktree is false and Any() is true; and that a registered worktree reads StrayPath false.
    origin: lens:code-tests
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/battencli/beamland.go:127
    summary: the beam-land member listing now reads a pair through PairWorktreeStands, and no test feeds it a non-worktree directory
    fix: Add a row to the existing beamMembers test that plants a plain directory at a member's task worktree path and asserts that member's HasPair is false, and one with a registered worktree asserting true.
    origin: lens:code-tests
  - id: F5
    severity: LOW
    class: design
    location: internal/fabricengine/remove.go:230
    summary: the respawnPairPush skip for ErrStrayPathKept, the strayErr versus out.err precedence on the remote returns, and the RemoveStrayTaskPath slug and prime-slug refusals have no test row
    fix: In the remove_leftoverdir_integration_test.go kept-directory test, assert no pair push was respawned. Add rows to TestRemoveStrayTaskPath for an invalid slug and the prime slug, each asserting an error and that nothing was deleted. Add a Remove row with a remote step failure and a kept directory asserting the remote step's error wins.
    origin: lens:code-tests
  - id: F6
    severity: LOW
    class: design
    location: internal/battencli/cancel.go:667
    summary: cancel --remove deletes a stray directory with no sign in its report, and nothing enforces that a pair ever stood at the path
    fix: Have removePairAndBranch record the deleted directory in the cancel report, as a field or a line the cancel output prints, replacing the warning it dropped. This needs RemovePair's result to carry whether the stray delete removed anything. That bool is already returned by the removeStray seam, so thread it into fabricengine.RemoveResult or pairteardown's result.
    origin: lens:code-correctness
---

### [BLOCKING] RemoveStrayTaskPath bypasses the destruction gate with no PATTERN amendment

**Location:** internal/fabricengine/destroy.go:1415, pattern/PATTERN-fabric-git.md (Destruction chokepoint)
**Issue:** `PATTERN-fabric-git` requires every executor in `destroy.go` to check containment, ownership, dirtiness and force, in that order. Its only exception is `removeWorktreeLeftover`, which takes ownership and dirtiness from the gated `git worktree remove` its caller just ran on the same target. The new `RemoveStrayTaskPath` calls `removeWorktreeLeftover` with no such git run. Its ownership proof is the caller's own knowledge, which no code checks. It has no dirtiness step, so Worktree-Teardown with `Force: false` deletes whatever stands at the path. The diff widened the `removeWorktreeLeftover` doc comment ("such as a `git worktree remove`") to fit, but left the PATTERN text unchanged, and no commit carries a `Pattern-Departure:` trailer. This answers the focus question on the precondition: it is assumed, not enforced. The only guards are containment under the hub root, the registered-worktree refusal and the `.git` refusal.

### [MEDIUM] teardownRequest's ClearStray is untested

**Location:** internal/battencli/wire.go:408
**Issue:** Only cancel_test.go asserts `ClearStray`. Dropping it from `teardownRequest` fails no test, and the resumed Worktree-Teardown that clears a refilled directory rests on it.

### [MEDIUM] PairParts.StrayPath has no test through the real inspectPair

**Location:** internal/fabricengine/pairparts.go:25
**Issue:** The cancel tests use a fake `PairParts`. A broken mapping from `inspectPair` or a missing term in `Any()` would break `cancel --remove` on a stray-only slug with every test green.

### [MEDIUM] Beam-land member listing change is untested

**Location:** internal/battencli/beamland.go:127
**Issue:** The change replaces `pairPresent` with `PairWorktreeStands` so a leftover directory no longer reads as a pair in the landing refusal. No test in the diff feeds `beamMembers` a non-worktree directory.

### [LOW] Error-path branches of Remove and RemoveStrayTaskPath have no rows

**Location:** internal/fabricengine/remove.go:230
**Issue:** The skip of `respawnPairPush` for `ErrStrayPathKept` and the order of `strayErr` against `out.err` on the remote returns are unpinned. The summary admits the `localMode` return has no test. `TestRemoveStrayTaskPath` has no row for the slug validation or prime-slug refusal that guard the delete's target.

### [LOW] cancel --remove deletes a directory without telling the operator

**Location:** internal/battencli/cancel.go:667
**Issue:** The old cancel warned of the directory and told the operator to save anything worth keeping. Now `cancel --remove` deletes it, and only a `logger.Info` line in `pairteardown` records it. A slug that never had a pair but has a directory of that name under the hub loses it with no mention in the report. The Goal licenses the delete, so this is LOW.

## Rejected

- Goal part 2, the board's "has a pair" check not changed: `internal/boardcli/cli.go:148` already wires `PairExists` to `PairWorktreeStands`, and `taskWorktreePresent` (`internal/battencli/wire.go:266`) already reads a non-worktree directory as absent, so the diff owes the board package nothing. The summary's claim that main fixed it holds.
- RemoveStrayTaskPath takes no teardown lock (lens:code-correctness): speculative. The lens did not confirm a window, and both callers hold their own teardown and cancel locks; the delete touches a local directory, not the slug's pair or origin branches that the lock entry covers.
- `leftPartNames` ignores `StrayPath` (lens:code-affected-set, unread): `Add` already refuses the stray path by name, so a start is still refused.
- Worktree-Teardown's `.git`-entry refusal has no resume way forward (handler): it is batten's own row, not a webster, shed or loom refusal, and the operator resumes after clearing the directory.
- Summary claims: the claims I checked match the diff, and the summary states honestly which paths ran no end-to-end check.
