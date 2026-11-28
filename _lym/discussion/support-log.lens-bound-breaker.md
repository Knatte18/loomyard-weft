```yaml
lens: bound-breaker
outcome: reported
```

Lens bound-breaker: each claim, bound and ordering promise in the record, with a counterexample where one breaks.

**Findings**

1. **BLOCKING (D8, D6): the `landed` wake-up costs a full gate under the lock for each run.**
   D8 sends a `landed` run down "today's already-landed path". That path is `Finalize.Call`'s locked pass, which runs `mergeAndGate` with no held shortcut. `mergeAndGate` is `mergeParent` then `gateAndFix`. The parent now holds the other runs' landings, so the merge writes a new tree into the run's task worktree, and the run gates tiers 0 to 2 on it.
   - Counterexample: a batch of A, B and C passes and lands. B then takes the lock, merges the new tip and gates for 10 to 15 minutes. C does the same after B. The serial gates the entry set out to remove come back.
   - Evidence: `finalize.go` Steps 2 to 6 loop calls `mergeAndGate` unconditionally. `mergeAndGate` is `mergeParent` + `gateAndFix`. The Call doc says the already-landed path "proceeds as for a fresh landing".
   - Fix: on `landed` with its commit on the parent, the run merges and gates nothing. It only records its landing and ends.

2. **BLOCKING (D7, its acceptance criterion): outcome `failed` can never be set, and the D7 criterion contradicts the mechanism.**
   D3 puts the leader's own entry first, and every halved prefix keeps the first entry. So a prefix of one is always the leader's own, and "a failing prefix of one that is another run's entry" never happens.
   - Counterexample: batch A, B, C, D with C bad. A, B, C, D fails. The first half, A and B, passes and lands. C and D stay queued with no outcome.
   - The criterion expects C to get `failed` while D stays queued. Under the record's own rules, C and D both stay unresolved.
   - Fix: either bisect the remainder within the same lock hold, or drop `failed` and rewrite D7's criterion.

3. **MEDIUM (D6, D8, D3): a stale `landed` outcome strands its run.**
   A leader that dies between writing the `landed` outcomes and the fast-forward leaves outcomes whose commits the parent lacks. The run that takes the lock next may be one of those members.
   - D3 makes a run leader only "with its entry still unresolved", and skips entries "whose outcome is already set".
   - D8 has no case for `landed` with its commit off the parent.
   - D6 lets only "a leader" clear such an outcome.
   - Result: that member is neither leader nor landed, so its behaviour is undefined.

4. **MEDIUM (D2, D6): a queued run's driver restart loses its landing repair.**
   A member's driver can restart while it is queued or after it was batch-landed: a crash, a resume, or the binary re-exec. Its old pid is then dead, and D2 has the next leader remove the entry, `landed` outcome included.
   - The restarted run finds the parent holding its work but has no entry. D6's ledger repair is keyed on the entry's planned commit, which is gone.
   - It falls into today's held path: a full gate (finding 1), then `recordLanding` with the parent's HEAD (`HeadSHA`). That HEAD is the chain tip, not the run's own landing commit, so the landing is recorded at the wrong tip.
   - Fix: keep an entry with outcome `landed` until its own run reads it, or key the repair on the `Lym-Landing` trailer on the parent's first-parent chain.

5. **MEDIUM (D6): nothing handles a parent head that moved between the chain build and the fast-forward.**
   D6 fast-forwards "with today's owner checks and lock re-read", but D1 removes the only head comparison, the lost-race check. The parent pusher's reset to origin and its settling of a refused parent are paths outside the batch that can move the local parent. The record does not say they hold the parent lock.
   - If the head is not the chain's base, the fast-forward either fails with no stated way forward, or a forced update silently drops commits.
   - Fix: land only when the parent's head equals the chain base; otherwise rebuild the chain and gate again, or halt with a way forward.

6. **LOW (D7): a culprit can cost extra gates.**
   Ticket order and join order can differ, for example a Front ticket from a run retaking a lock it freed. The culprit can then be re-batched behind a run it delays, costing gates beyond the "about log2(n)" the rationale claims. That claim holds per batch, not per culprit.

**Claims attacked**

- D1 "merge stays lock-free, so no conflict session holds the lock for a clean run": held.
- D2 liveness by pid and start: held for the wait itself; broken on restart (finding 4).
- D3 skip rules and leader election: broken by a stale outcome (finding 3).
- D4 chain shape and trailers, and `MergeTree` taking commits: held. Fabric's `HoldsWork` merge-tree test reads a squash chain as holding the work.
- D5 detached gate, inert-path check against P, verify and green caches, and restore on resume: held.
- D6 "no landing lost after a leader dies": broken (findings 3, 4). Fast-forward ordering: broken (finding 5).
- D7 halving: `failed` unreachable (finding 2). Gate count: weaker than stated (finding 6).
- D8 `landed` path: broken (finding 1). `conflict` path: held.
- D9 retired-key warning: held, per `PATTERN-config-layers`' retired-key line.
- D10 pull-request exclusion, with its GitHub-rule bound: held.
- Constraint "a live verify keeps the lock": held during a gate. There is a gap of no live verify between halving gates, but `parent_lock_idle_release_min` is 10, so it held.

**Tool calls**

- `grep` of `parentHoldsWork` uses in landingshed: to find where the already-landed path branches, which the context lacked.
- `finalize.go` 260–300, 365–380, 468–500, 1505–1540 and fabric's `HoldsWork`: for the held path's behaviour and the holds-work test, not in context.
- `grep` around lines 500–560 for held, gate and `mergeAndGate`: to see whether the held path gates, not in context.
- `finalize.go` 518–534: the condition before `mergeAndGate` under the lock, not in context.
- `grep` of `mergeAndGate`'s body: to confirm it has no held or already-gated shortcut, not in context.
