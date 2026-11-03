```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I challenged each proposal the fixer carried out as proposed, and the own changes they depend on.

1. **MEDIUM**: D2's Land way forward on the not-live sentinel restores from "the one whose commit is newest".
   - **Summary:** this can pick the wrong tip, because the fixed D4 and D6 now push two archive tags for one removal.
   - **Evidence:** D4 step 3 now also pushes "an archive tag at origin's tip when that tip, after step 2, is neither the local tip nor an ancestor of it". D6 extends the same rule to the sweep.
   - **Case:** a removal at Land meets an origin weft tip that another writer moved, and that tip diverges from this machine's tip. It pushes two `archive/<slug>/*` tags. The diverged origin commit can carry the newer committer date. The pre-run then names that tag, and `git branch <weft branch> <tag>` restores the other writer's tip instead of the local tip that holds this run's records. The D2 Land criterion ("with the child's run records present") then fails.
   - **Fix direction:** name the tag at the local tip D4 archived, which is not the same as the newest tag.

2. **MEDIUM**: D8 covers only a branch `Add` forks. D1 and D2 still re-adopt a local weft branch that origin lacks.
   - **Summary:** that branch can carry the same stale tracking ref, and nothing drops it.
   - **Evidence:** D8 says it drops the ref "when `fabricengine.Topology.Add` forks a branch … whose origin copy its pre-flight probe found absent". D2 Land says "The restored local weft branch is live by `Add`'s rule". The D1 bound says it re-adopts "a removal that left the local weft branch behind". In both cases origin lacks the branch, so `probeWeftLeftover` returns live with no tip.
   - **Case:** the Constraints evidence shows a tracking ref outliving a removal. In that case D1 step 4's second reconcile reads the re-adopted weft branch gone (`judgeBranch`: an empty `originHead` is `BranchGone`). The resume stops on that verdict, so the D2 Land recovery fails the same way the 2026-11-03 run did.
   - **Fix direction:** D8 should also cover adopting a local branch whose origin copy is absent.

3. **MEDIUM**: the F1 fix keeps "the run's parent pair as `Parent`", and the F5 own change makes every refusal outside the closed set transient.
   - **Summary:** an unforkable parent turns into an endless retry loop.
   - **Evidence:** `add.go:227-231` calls `resolveAddParent`, which refuses a parent that cannot be forked from before the weft probe. That refusal is not in D2's closed set, so it is transient: "dobby retries".
   - **Case:** a child run whose parent pair was removed after the child's pair went is refused on every pass and never reaches a failed stop. That is the "fails pass after pass" outcome the Goal forbids.
   - Also: with the weft live but the warp branch on neither side, `Add` forks the warp branch from the parent's or the prime's HEAD. D1 now says this "forks nothing", which the code does not hold for the warp side (`add.go:419-430`).

4. **MEDIUM**: D3's Finalize exception says the record "is left to the adopt that clears that pause".
   - **Summary:** no command is named, and the case may have no such adopt.
   - **Evidence:** in the adopter-first case another machine landed the work, so origin reads the entry as ended. `resumeRefusal` then points at `lym cleanup --apply`, not an adopt. `ownerHaltReason`'s adopts move ownership to this machine and do not clear a batten record.
   - **Case:** the `running` record sits under the owner pause with no stop or warning naming a step that clears it. The D3 criterion only checks that the cancel is absent, so the stop names no way forward for the record.

5. **LOW**: D5's rewritten rule only defines when a commit is present.
   - **Summary:** a verdict that a branch is absent has no outcome under the new text.
   - **Evidence:** the rule says "a verdict that rests on a commit or branch being absent … The commit the stop names is present when … otherwise it is unsettled, and never read as absent."
   - **Case:** `ls-remote` returns no line for the ref. The rule gives no answer for whether the branch is absent, so the tackler cannot settle the branch-gone verdict the stencil change targets.

I made no further tool calls; my context already held everything this lens needed.
