```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the code and tests changed for F1, F2, F5 and F7 for correctness, by running one concrete input through each.

**Findings**

- LOW. The comment above `CreateWorktree` in `internal/battencli/wire.go` (the lines 939-940 text) says the claim, probes, reconcile and push run outside the hold "so a create waiting on origin holds up no other run's create". The F4 fix says `Add` still does the adopt fetch, the weft fast-forward fetch and the archived-leftover replace under the hold. A create doing one of those waits on origin while holding the lock, so the sentence is slightly too broad. The F4 fix text named this comment only "if it states the same". It does not use the words "network call", so leaving it is defensible. This belongs to the consistency lens too.

**Traces with no defect found**

- **F1, `add_hold_test.go`:**
  - `HeldAroundTheAdd`: `hold` runs before any worktree write, so `existsAtHold` is false. The deferred release runs after the writes, so `existsAtRelease` is true.
  - A hold moved after the writes would set `existsAtHold` true and fail the test.
  - A hold released early would set `existsAtRelease` false and fail the test.
  - `FailedHoldMakesNothing`: `hold` returns the error, nothing is made, and the `wantMade` and `existsAtHold` checks hold. `existsAtRelease` is not asserted there, correctly, since no release exists.
- **F2, `battencli` tests:** the tier-1 gate passed.
  - Lifecycle test: the first create gives takes/releases 1/1, because the board has no beam for the slug and `refuseHeldMember` returns before its hold. The re-entry over the complete pair adds no take.
  - `late`: `Add`'s hold plus the removal's hold gives 2/2.
  - `crashed`: the complete pair skips `Add`, so the removal takes the only hold, 1/1.
  - If the closure stopped passing the hold to `Add`, the lifecycle and `late` rows would fail.
  - `TestRefuseHeldMember` failing-hold row: `refuseHeldMember` returns `err` unwrapped from `holdPrime()`, so `errors.Is(holdFailure)` holds and the count is 1. A removal attempted before the hold would reach `pairteardown.New` on a temp hub and almost certainly return a different error, so the sentinel check catches it, though "no removal ran" is never asserted directly.
  - The other three rows have no hold error and want 0 takes.
- **F7, `create_test.go`:** two `hold()` calls with a release between them give 2 `Acquire` calls and 2 releases. A `sync.Once` shared across holds would give 1 release and fail the test. `Sleep` is never reached because `Acquire` succeeds at once. `for range 2` compiled under the gate.
- **F5, F6 and the other F4 edits:** comments only, with no signature or behaviour change, so no caller is affected.
- **Gate:** tier 0 and tier 1 over `fabricengine`, `battenshed` and `battencli` passed with 0 failures.

**Tool calls:** none. The context already held every changed file and the gate results.
