```yaml
lens: code-scope
outcome: reported
```

Lens code-scope: tracing each change in the diff to the Goal points of the board entry.

**Decisions.** These are the Goal points under `## Idea` in `_lym/discussion/decision-record.md:14-25`. The record has no `Added after Discussion` entries.
- G1: merge the parent's tip into the task worktree and gate the merged tree, unlocked.
- G2: take the parent lock and check that the parent's tip is still the commit the gated tree was merged from.
- G3: if it is, land at once, with the lock held for seconds.
- G4: if the parent moved, release the lock, merge the new tip and gate again, unlocked.
- G5: after a bounded number of lost races, fall back to merging and gating under the lock.
- G6: no drift; what lands passed the gate on top of the parent it lands on.
- G7: the fixer and fix review already run with the lock freed; the change is to the merge and the gate around them.

**Per change, what it traces to:**
- C1: `gateUnlocked` and the reworked `Call` loop, which resets `gatedTip`, runs the unlocked pass while `lostRaces` is below the bound and goes round again on `parentMoved` (`internal/landingshed/finalize.go:309-357`). Traces to G1 and G4.
- C2: step 1b4b, the head compare against `gatedTip` that sets `parentMoved`, returns nil and frees the lock through the deferred release (`finalize.go:440-452`). Traces to G2, G4 and G6.
- C3: the lost-race counter and its fallback, `FinalizeLostRaces` and `finalizeLostRacesKey`. This includes the `ConfigSpec` category, the `LoadConfig` check for values below 0, the template line and the new row in `TestLoadConfig_InvalidKeys` (`config.go:77-80,178-179,254-256`, `keynames.go:470`, `template.yaml:546`, `config_test.go:53`). Traces to G5.
- C4: `parentMergeTip` taken out of `ParentAlign.mergeTip` and shared, plus the ParentAlign call-site change (`parentalign.go:482-537`). Forced by C1, which needs a tip picked with no lock held.
- C5: the new `tip` parameter on `mergeParent` and `judgeParentMerge`, with `keepParentHead` moved into `mergeAndGate` (`finalize.go:1002-1008,1294-1362`). Forced by C1, so the unlocked merge can use the picked tip while the landed-diff base is still read under the lock.
- C6: `lockHeld`, and `freeLock` freeing only a lock this process holds (`finalize.go:83-84,393-397,545-552`). Forced by G7 together with C1: a fixer or fix review that starts in the unlocked pass must free nothing.
- C7: a parent that already holds the task's work runs no unlocked pass (`finalize.go:335-341`). Forced by C2: that path skips the parent sync, so the head compare cannot be judged there.
- C8: prose and comments, in the file header, the `Finalize` struct fields, `Call`, `landUnderLock`, `mergeParent`, `doc.go`, the `deps.go` `ConfigChanges` comment, `docs/overview.md` and `contracts/orchrole/role-landing-halts.md`. Forced by C1 to C7, which make the old text stale.
- C9: the new test file `finalize_unlocked_test.go`. Forced, as the test of C1 to C3 and C7.
- C10: commit dd3aea19e, which breaks wrapped comment lines at sentence boundaries. Forced by the comment lint over C8.

**Per decision, what builds it:**
- G1: C1, with C4 and C5.
- G2: C2.
- G3: C2, plus the existing skip of an already verified tree in `verifyGate.check` (`verifygate.go:261-264`), which the locked `mergeAndGate` reaches when the head matches.
- G4: C1 and C2.
- G5: C3.
- G6: C2 and C7.
- G7: C6.

**Findings: none.** Every change traces to a Goal point or is forced by one that does. Every Goal point is built by at least one change. No commit in the range carries a `Pattern-Departure:` trailer, and none was needed.

One thing outside this lens, for the affected-set lens: the Finalize comments in `contracts/recipes/{sampler,darn,tapestry,hardener}-recipe.yaml` still say "Finalize takes the parent lock first, merges the parent…". The diff leaves them untouched.

Tool calls: none.
