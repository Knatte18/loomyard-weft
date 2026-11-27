```yaml
lens: fix-consistency
outcome: reported
```

Task: consistency check of the fix's change set against its siblings, twins and cross-references.

1. **MEDIUM**, `internal/landingshed/deps.go`, `ConfigChanges` doc ("so the diff's base is the task's fork point unless an earlier row merged the parent in").
   - The bound is false for a re-entered Finalize. An earlier `Call`'s lock-free merge stays on the task branch, for example after a gate halt, a spent fixer or an owner halt under the lock.
   - The next `Call` reads the notice (step 1b0) after that merge, so `merge-base(parentTip, tip)` is the parent tip merged then, although no earlier row merged anything.

2. **LOW**, twins of the F10 `freeLock` rewording that were left unchanged.
   - `freeLock` now says the run "runs its passes again, the lock-free pass first while lost races remain".
   - The `Call` doc still says: "Once their change is let through, Finalize takes the lock again at the front of the queue, and it runs every step under the lock again, from the pull-request step on."
   - `docs/overview.md` (about lines 1386-1389) still says: "It then retakes the lock at the front of the queue. It merges the parent anew and verifies under the lock before it lands".
   - Both leave out the lock-free pass that `Call`'s loop runs first.

3. **LOW**, `internal/landingshed/doc.go`, the inserted sentence.
   - It reads "It runs the owner checks before that merge, and again under the lock." and is followed by "It then takes the parent lock through the shared guard."
   - The text now reports checks under the lock before it says the lock is taken, and "then" follows the wrong step.

4. **LOW**, comment lines over the 120-character limit, by my count and not measured.
   - The "After landing.finalize_lost_races lost races, and over a parent already holding the work, …" line in the darn, sampler, tapestry and hardener recipes is about 125 characters.
   - The new `gateUnlocked` comment "The pre-push check leaves a run at this row alone, so the owners are judged …" in `finalize.go` is about 121.

I found no other disagreement in `prepush.go`'s exemption sentence, `role-landing-halts.md`, `orch-role.md`'s owner-halt paragraph or the `PATTERN-owner-machine` hits the grep showed. `tapestry-rows-before-motif.yaml` keeps the old comments by its own header's rule, as the parent noted.

Tool calls:
- `git grep` for config-notice, owner/reconcile and under-lock wording: my context held only the earlier lock-wording search.
- `sed` over `orch-role.md`, `role-landing-halts.md` and `doc.go` lines 1-24: my context had not shown those spans.
