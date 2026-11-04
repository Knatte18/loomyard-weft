---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/add_hold_test.go:25
    summary: "No test pins that Add's hold is taken before and spans the worktree writes, so moving the hold to after the mutations would still pass"
    fix: "In internal/fabricengine/add_hold_test.go, in the HeldAroundTheAdd row of TestAdd_HoldsTheCallersLock, make the hold closure record whether fabricengine.WorktreePath(h.Location, slug) exists when Hold is called (want false) and make the release it returns record whether that path exists when it runs (want true), then assert both after Add returns. Do not add a probing Add test unless hubforge.CopyHub already provides a reachable origin."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/battencli/wire.go:941
    summary: "No test asserts the create closure hands its hold to Add or takes it around a held member's pair removal, because every battencli test passes holdNothing"
    fix: "In internal/battencli, replace holdNothing with a counting hold in three places. First, in lifecycle_integration_test.go TestBattenIntegration_RunReconcilesTheHubBeforeAndThePairAfterCreating, assert one take and one release after the first CreateWorktree (the hold reached Add and was released) and no further take after the second call over the complete pair. Second, in beamfork_integration_test.go, make the held-member rows MemberWhoseBeamHoldsAfterItsPairIsMadeLosesThePair and MemberResumedOverACompletePairWhileItsBeamHoldsLosesThePair assert the takes and releases around the removal (the complete-pair row takes exactly one). Third, in beamfork_test.go TestRefuseHeldMember, make holdPrime a per-row field and add a row where the beam is held and holdPrime returns a sentinel error, asserting errors.Is(err, sentinel) and that no pair removal ran."
  - id: F3
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/landing/summary.md
    summary: "The diff narrows only the prime lock's span; the Goal's one fetch and one remote-refs read per create, no synchronous push on the create path and the Open-Window delay are left unmade with no recorded decision"
    fix: "Leave the code in place; the fixer changes nothing for this finding. The run's parent records or refuses, with lym loom decision add, the Goal ideas this change leaves unmade: one fetch and one remote-refs read per create, no synchronous push on the create path (the board claim's Bolt push stays synchronous in this diff), and the Open-Window delay that summary.md says is not established. Defer this finding to the parent."
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: internal/battenshed/create.go:35
    summary: "Docs say the hold is never taken around a network call, but Add still runs the warp adopt fetch, the weft fast-forward fetch and the archived-leftover replace under it"
    fix: "Reword the claim to name the origin probes instead of every network call in three places: the HoldPrimeLock doc comment in internal/battenshed/create.go (the words 'and never around a network call'), the Worktree-Create paragraph in internal/battenshed/doc.go (the words 'never around a network call'), and, if it states the same, the comment above CreateWorktree in internal/battencli/wire.go. State that Add's adopt fetch (add.go:442), weft fast-forward fetch (add.go:503) and archived-leftover replace (add.go:661) still run under the hold."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/add.go:62
    summary: "Add calls Hold under the slug's teardown lock, the reverse of Worktree-Teardown's prime-then-teardown order, and the premise that keeps this from stalling is recorded nowhere in code"
    fix: "In internal/fabricengine/add.go, extend the AddOptions.Hold doc comment with one sentence: Add calls Hold while it holds the pair's teardown lock, so a Hold that waits on a lock whose holder is waiting for this pair's teardown lock stalls both until the teardown wait runs out; it is safe only because a pair is never created and removed at once."
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/battenshed/primelockwait.go:1
    summary: "The file header says both prime-lock producers call acquirePrimeLock, but Worktree-Create now reaches it only through its HoldPrimeLock"
    fix: "In internal/battenshed/primelockwait.go, reword the file's first comment line to say acquirePrimeLock is the bounded, cancellable wait that Worktree-Teardown calls and that Worktree-Create's HoldPrimeLock calls on each hold, in place of a single Acquire."
  - id: F7
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/battenshed/create_test.go:72
    summary: "No test covers two sequential holds in one Call, the production case of Add's hold followed by refuseHeldMember's hold"
    fix: "In internal/battenshed/create_test.go, add a subtest HeldAgainAfterRelease to TestWorktreeCreate_TakesThePrimeLockOnlyThroughItsHold: the closure calls hold, releases, calls hold again and releases again. Assert the Done outcome, two Acquire calls and two releases."
---

### [MEDIUM] No test pins that Add's hold spans the worktree writes

**Location:** internal/fabricengine/add_hold_test.go:25
**Issue:** `TestAdd_HoldsTheCallersLock` counts takes and releases only after `Add` returns, and runs with `SkipGit`, so no probe goroutine starts. A hold moved to after the worktree and branch writes still passes, although the hold's purpose is to serialize those writes. The ordering against the probes and the probe wait on every return are not tested either; they are left out of the fix because they need a reachable origin.

### [MEDIUM] No test pins the create closure's use of its hold

**Location:** internal/battencli/wire.go:941
**Issue:** Every `battencli` test passes `holdNothing`, which never asserts a take. Nothing shows that the closure hands the hold to `Add`, that the claim and `reconcileTaskPair` run outside it, or that `refuseHeldMember` takes it around a held member's pair removal and returns a hold error. `TestRefuseHeldMember` only asserts the hold is not taken for the non-removal cases. This is the behavior the commit exists for.

### [MEDIUM] The Goal's other ideas are left unmade with no recorded decision

**Location:** _lym/landing/summary.md
**Issue:** The Goal asks to let creates run side by side, cut the network round trips to one fetch and one remote-refs read per create, keep a synchronous weft push off the create path, and find why Open-Window takes over a minute. The diff makes only the first, by narrowing the prime lock. The summary discloses the remaining round trips, the remote-refs queue and the unexplained Open-Window delay under Known limits, and keeps the claim's Bolt push synchronous on purpose, citing `PATTERN-owner-machine`. The decision record carries no `Added after Discussion` entry for any of it. The commit message and summary do not claim more than the diff does. The only sound change is the parent's `lym loom decision add`, so the code stays.

### [LOW] Docs claim the hold spans no network call

**Location:** internal/battenshed/create.go:35
**Issue:** `HoldPrimeLock`'s doc comment and the Worktree-Create paragraph of `doc.go` say the lock is never taken around a network call. `Add` holds it to its return, and on an adopt it runs `FetchWithArgs` at `add.go:442`, on a behind weft branch the fast-forward fetch at `add.go:503`, and for an archived leftover `deleteRemoteBranch` at `add.go:661`. Only the origin probes are outside it, and a fresh create makes no such call. The `docs/overview.md` line names the probes only and is accurate.

### [LOW] The lock order against Worktree-Teardown rests on an unrecorded premise

**Location:** internal/fabricengine/add.go:62
**Issue:** `Add` takes the slug's teardown lock and then calls `Hold`, which takes the prime lock. `Worktree-Teardown` takes the prime lock and then the slug's teardown lock. A create and a removal of one slug that overlap each wait on the lock the other holds until `teardownWait` runs out (`teardownlock.go:124`). The waits are bounded, so this is a stall and no deadlock, and the premise that a slug is never created and removed at once is stated only in the summary.

### [LOW] primelockwait.go header names the old callers

**Location:** internal/battenshed/primelockwait.go:1
**Issue:** The header says both producers call `acquirePrimeLock`. Worktree-Create now calls it from the `hold` closure in `create.go`, possibly several times per run.

### [LOW] No test covers a second hold after the first is released

**Location:** internal/battenshed/create_test.go:72
**Issue:** `Add`'s hold, released at its return, is followed by `refuseHeldMember`'s hold in the same `Call`. Each hold gets its own `sync.Once`, so the code is correct, but the new subtests cover a single hold only. The lens also asked for a test that wraps the hold error before it reaches the row; it is rejected below.

## Rejected

- Probes now start before the pre-flight refusals, so a refusal such as `ErrSourceDirty` returns only after `probes.Wait` and can wait on a degraded origin (code-correctness note): the summary and the `Add` doc comment state that every return waits for the probes, no result is wrong, and the lock is released before the wait. Moving refusals above the probes would change the precedence the summary promises to keep.
- A test that wraps the hold error with `%w` before it reaches the row (code-tests F-C, second half): `Add` returns the hold error as is, which `FailedHoldMakesNothing` pins with `errors.Is`, and `createRefusal` passes unknown errors through unchanged, so a wrapped case is not a behavior of this diff.
- Probe ordering and the probe wait on every `Add` return (code-tests F-A, probe part): a test needs a reachable origin, which `TestAdd_HoldsTheCallersLock` does not use; the ordering is held by the code's own defers, which the correctness lens traced as correct.
- The board entry could not be read with `lym board get` (invalid JSON error), so the decision record's `## Goal` stood as the entry text; no finding rests on the board copy.
