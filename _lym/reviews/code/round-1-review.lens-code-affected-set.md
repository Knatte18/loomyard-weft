```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens over the one-row test change in `internal/creelengine/deliver_test.go`.

**Per-change list**

1. The question's sender changes from `strand(beta)` to `strand(orch)`, in the row at `deliver_test.go:2618`.
   - Set found:
     - The `chase` id is found through `creelPostTo(e, creel, alpha)`, which filters on `From: creel` and `To: alpha`. It does not depend on the question's sender.
     - The `which answers #1` assert depends on the question's id being 1. It is the first post in a fresh env, and `ChaseOf` is the question's id.
     - Strand `orch` is already used as a sender in the same table at `deliver_test.go:2426`, with `Hold(strand(orch), alpha, time.Hour)` at line 2429.
     - The chase body names `msg.From`, but the row does not assert on the body.
   - Covered: yes.

2. A new hold by orch on alpha, in the same row.
   - Set found:
     - `Hold` at `verbs.go:889` permits the call. `strand(orch)` is the privileged sender, the target alpha is not an orch name, and the same call already runs at `deliver_test.go:2429`.
     - `ageChases` at `chase.go:129` skips a message only when the hold owner differs from `msg.From`. Here the owner is orch and `msg.From` is orch, so the clock runs.
     - `snapshot` at `deliver.go:541` leaves out mail whose sender is not the hold owner, so the chase from creel is not typed.
     - `remind` at `deliver.go:405` returns without a write when `unread` is empty.
     - The notices and injects paths are not reached.
     - The refused `Forward` at `verbs.go:581` still reaches its `ChaseOf` case before any hold check.
     - Inbox is not filtered by the hold. `verbs.go:371` and `claimableLocked` filter only the claim path, not `Inbox`.
     - The `Hold` call adds a `held` journal entry, but it lands before `before` is read, so the count is unaffected.
     - `maxHoldDuration` permits the one-hour hold, and the `ADormantHolder…` row already uses it.
   - Covered: yes. The row ran 20 of 20 under the gate with 0 failures (parent's run).

3. Docs and rules.
   - Set found:
     - There is no `doc.go`, `docs/overview.md` or `PATTERN.md` change to make, because this is a test-only change.
     - `PATTERN-test-speed` is met: the injected clock and `t.Parallel` on `TestScheduler_Chases` are kept, with no sleep added.
     - The commit message has no `Pattern-Departure:` trailer, and none is needed.
     - No PATTERN entries are declared as changed.
     - The summary's claims match the diff.
     - The added comment is above the code, not end-of-line, and follows the comment convention.
   - Covered: yes.

**Findings:** none.

**Tool calls:** none. My context already held everything the lens needed.
