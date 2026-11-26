```yaml
lens: fix-proposal
outcome: reported
```

Task: `fix-proposal` lens, challenging the reviewer's proposals as the fixer carried them out.

**Findings**

1. **MEDIUM (arguably BLOCKING)** — F2: `NewVerifyGate`'s takeover of the persisted pre-fix head (`earlierPreFixHead`, the `readEarlier` block in `verifygate.go`).
   A restarted gate now applies `fixRejection` to everything committed between the runs, not only the earlier fixers' commits.
   - **Case:** run 1 fails the gate and ends stuck. The stuck reason says "commit a fix on the task branch; lym webster run". The parent's fix includes a parent merge-in with a hand conflict resolution, or any other merge that isn't a clean parent merge.
   - **Before:** the new closure had `preFix == ""`, ran no commit check, and passed on a green verify.
   - **Now:** the persisted head is on HEAD's history with no batch above it, so the first arrival fails `Terminal`.
   - **No way out that keeps the work:** a Terminal failure never reaches `recordPass`, so `PreFixHead` stays and every later run fails the same way. The only way forward it names, `reset --to pre-fix`, discards the hand fix.
   - **Related:** after a restart, `reset --to pre-fix` also targets the older base, so it drops the parent's between-run commits.

2. **MEDIUM** — F3: the join rule in `recordGatePass` (`if fixedBase != st.FixedHead`).
   It joins two vetted ranges only on exact equality, though `accept-audit` treats clean parent merges as harmless everywhere else.
   - **Case:** run 1 passes with fixes (`FixedBase = lastHead`, `FixedHead = H1`). A clean parent merge M lands on H1. Run 2's gate fails at M, is fixed, and passes at H2.
   - **Result:** `FixedBase` becomes M. `walkParentMerges(lastHead, M)` reaches run 1's non-merge fix commits and refuses, although both gates vetted every non-merge commit in the range. The review's third bullet (accept the commits the gate passed at) fails for this history.

3. **LOW** — F14 with F1: the `Ignored` row in `TestAcceptPendingAudit_RecordsSidePath`, and the `IgnoredPath` branch in `recordsSideStatus`.
   The row passes with the branch deleted. An ignored file is absent from the base commit and present now, so the blob comparison already returns "changed". An absent path is never reported as ignored, and a tracked file isn't either. The branch can no longer change an outcome, and the row pins nothing about it.

4. **LOW** — F4: line 308 of `webster-template-master.md`.
   The proposed wording says "your two contract files and the friction note" with no condition. With friction off, the prompt names a note that doesn't exist. Lines 319 and 321 both say "when one is named".

Tool calls used: none.
