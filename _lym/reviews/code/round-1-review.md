---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/battencli/owner_test.go:170
    origin: lens:code-tests
    summary: No test pins that a Worktree-Teardown resume with the task worktree present still runs the reconcile, so dropping the worktree-absent condition of the skip would pass every test
    fix: In TestOwnerPreflight_TeardownWithPairGone add a row with producer battenrecipe.NameWorktreeTeardown, an owner fake showing the run owned here, and the task worktree directory created before the call, expecting the reconcile to run. The table needs a per-row flag for creating the worktree, and a fixture that makes taskWorktreePresent true; for the call c.location.HubPath-based path fabricengine.WorktreePath(c.location, c.slug) must hold a resolvable worktree (a directory with the weft-root .lym-anchor that lymcwd.ResolveWorktree accepts).
  - id: F2
    severity: MEDIUM
    class: consistency
    location: pattern/PATTERN-owner-machine.md:20
    origin: lens:code-affected-set
    summary: PATTERN-owner-machine states that every hand pass is a resume and gets the origin reconcile, and that acting on a run starts with it, but a Worktree-Teardown resume with its pair gone now skips it
    fix: In pattern/PATTERN-owner-machine.md, add the exception to the passes bullet at line 20 and to the "Origin reconcile first" section; a resume at Worktree-Teardown whose task worktree is gone runs no origin reconcile, judges the owner on the local board alone and writes no origin-owed marker. Update the one-line PATTERN-owner-machine entry in PATTERN.md only if its text no longer holds.
  - id: F3
    severity: MEDIUM
    class: consistency
    location: internal/battenshed/doc.go:59
    origin: lens:code-affected-set
    summary: The package doc describes a resume as always running the run-scope origin reconcile first and the local board as a fallback only for unreachable origin, which the diff makes untrue for a Worktree-Teardown resume with the pair gone
    fix: In the resume paragraph of internal/battenshed/doc.go, add one sentence after the unreachable-origin sentence saying that a resume at Worktree-Teardown whose task worktree is gone runs no reconcile and judges the owner on the local board alone, so the teardown retries what is left of the pair.
  - id: F4
    severity: LOW
    class: design
    location: internal/battencli/owner_test.go:173
    origin: lens:code-tests
    summary: The first D1 acceptance criterion says owned here or unowned, but the table has only an owned-here row on the skip path
    fix: Add a Worktree-Teardown row with owners ownedBy("here", "", true) that expects a resume, no reconcile and no origin-owed marker.
  - id: F5
    severity: LOW
    class: consistency
    location: internal/battencli/owner_test.go:3
    origin: lens:code-affected-set
    summary: The file header says the file covers the start check and the status owner keys, and omits the teardown pre-flight test and the refusal exit codes it now holds
    fix: Extend the file-level comment on line 3 to name the Worktree-Teardown owner pre-flight and the pass exit codes.
---

### [MEDIUM] Missing test for a Teardown resume with the pair present

**Location:** `internal/battencli/owner_test.go:170`
**Issue:** The second acceptance criterion requires that a Teardown resume with the task worktree present still reconciles.
The only reconcile row uses `NameLand` with the worktree absent, so removing the `!present` check in `ownerPreflight` (`owner.go:240`) breaks no test, and `newFakeReceiver` builds no task worktree.

### [MEDIUM] PATTERN-owner-machine contradicts the new skip

**Location:** `pattern/PATTERN-owner-machine.md:20`
**Issue:** The rule says every hand pass is a resume and gets the origin reconcile.
The diff adds an exception and leaves the rule document untouched, so the repo's own rule now disagrees with the code.
The decision record cites this entry as the one the local-board judgement satisfies.

### [MEDIUM] battenshed package doc stale on the resume pre-flight

**Location:** `internal/battenshed/doc.go:59`
**Issue:** The doc describes the resume's reconcile as unconditional and the local board as an unreachable-origin fallback only.
`cli.go` and `arm.go` were updated for the new exception; this sibling description of the same pre-flight was not.

### [LOW] Unowned row missing on the skip path

**Location:** `internal/battencli/owner_test.go:173`
**Issue:** "Owned here or unowned" is one criterion; only the owned-here half has a row.

### [LOW] Stale test file header

**Location:** `internal/battencli/owner_test.go:3`
**Issue:** The header does not name the new teardown pre-flight test.

## Rejected

- Unwrapped `taskWorktreePresent` error on the new branch (code-correctness, LOW): it is the same bare passthrough as the existing Worktree-Create branch, fails closed, and a wrapping change would diverge from that sibling for no behavioural gain.
- No test row for a `taskWorktreePresent` failure (code-tests, LOW): the error is an unmodified passthrough of a helper with its own coverage, and the code-quality rule against testing misuse of internal calls applies.
- No test row for a receiver with no owner seams on the skip path (code-tests, behaviour 10): a receiver with no owner seams occurs only in tests, never in production, so the rule against defensive tests for internal states applies.
- Debris-only worktree directory blocks the teardown retry (code-correctness note): not a defect of the diff; the decision record's Open risks cover the neighbouring case and the Worktree-Create branch behaves the same.
