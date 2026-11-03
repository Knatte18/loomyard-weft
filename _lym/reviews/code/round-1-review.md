---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/battencli/wire.go:207-229
    summary: A stray directory still passes presentTaskWorktreePath's stat and fails in taskWorktreeLocation with the bare not a git repository error, so the new docs claiming every batten verb and one answer overclaim.
    fix: In internal/battencli/wire.go, make presentTaskWorktreePath call taskWorktreePresent(prime, slug) instead of os.Stat, return its error, and when it answers false return the existing by-name refusal (the same fmt.Errorf text, built from fabricengine.WorktreePath(prime, slug)), otherwise return the path. Then taskWorktreeLocation, taskWorktreeLocator, goto and the SpawnPush seam of readopt refuse a stray directory by name too. If that is not wanted, soften the sentence at internal/battencli/cli.go:69 about every batten verb reading it so, and the taskWorktreePresent doc line about batten's one answer in wire.go:253, to name the callers that use taskWorktreePresent.
  - id: F2
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/battencli/beamland_integration_test.go:107-120
    summary: No test drives hubStartGuard's real ReachedParent wiring through the reached case, so dropping that line leaves every test green while a marked task whose work reached its parent is refused again.
    fix: In TestBeamLanding_ReachesWorktreeCreateSkip, add a subtest beside MarkedPairOfATaskIsRefusedNamingTheCancel that calls refuse on hubStartGuard(h.Location, "promoted") with the slug promoted, the verb run, a shedengine.Status whose CurrentProducer is battenrecipe.NameWorktreeTeardown, and found and taskWorktreePresent both true, and asserts a nil error, which pins the real hubChildReachedParent wiring.
---

### [LOW] Docs claim one answer and every verb, but the stat-only locator remains

**Location:** `internal/battencli/wire.go:207-229`, `internal/battencli/cli.go:69`
**Issue:** `taskWorktreePresent` now reads a path that stands but does not resolve as absent, and its doc and the package doc say batten has one answer and every verb reads it so.
`presentTaskWorktreePath` still tests only `os.Stat`, so for a stray directory `taskWorktreeLocation` returns the path and then fails in `lymcwd.ResolveWorktree` with the bare "not a git repository".
Its callers `goto.go:19`, `readopt.go:78` and `readChildStatus` tolerate the error or are gated by `taskWorktreePresent` first, so no reachable path fails today, and the decision record's three Expected bullets are met.
The claim is stronger than the code, and the next caller of `taskWorktreeLocation` outside a gate meets the old failure.

### [LOW] The real ReachedParent wiring has no reached-case test

**Location:** `internal/battencli/beamland_integration_test.go:107-120`
**Issue:** `MarkedPairTaskWhoseWorkReachedItsParentPasses` stubs the `ReachedParent` seam, and the integration subtests cover only the not-reached task (`promoted`) and the pruned slug (`landed`, which never reaches the seam).
Removing `ReachedParent: hubChildReachedParent(location, slug)` from `hubStartGuard` leaves a nil seam that reads every run as not reached, and nothing fails.

## Rejected

- `taskWorktreePresent` discards every `ResolveWorktree` error, so an unreadable anchor reads as absent (code-correctness F1): the discard is the recorded design decision in the summary, matches what the landed `taskWorktreeGone` did, and fabric's add refuses the path by name.
- Stray beam directory wording in `beamfork.go` is unverified and untested (code-affected-set A2): `beamForkPrecondition` is already covered with `pairPresent` false, a stray directory only changes the input, and `lym beam create` then meets the add's named refusal, not a loop.
- Cleanup still failing with "not a git repository" (decision record bullet 4): batten cleanup resolves no task worktree in the code I read, so the three verbs the goal names are covered by the `taskWorktreePresent` change; the summary names this for a hand check.
