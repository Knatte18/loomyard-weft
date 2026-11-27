```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged each proposal the fixer carried out as written (F1–F6), looking for cases where the proposed change itself is wrong.

**1. BLOCKING — F3, card 11 step 1, the run read as owned elsewhere.**
Changed text: "A run read as owned elsewhere gets no `done` write and keeps its landing, and the pass writes one `held_back` record of the parent's branch through `RecordRefusal`, naming the run and `lym adopt` of its subject."
- **The case:** a landing's tip is already on origin, and the run was then adopted by another machine. Nothing is left to push for that run, and its new owner legitimately owns it.
- **What breaks:** the `held_back` record makes the landing target refused, because `TargetRefused` reads it through `PrimeCodeRefusalStands`. Card 11 says "A refused parent target starts no pusher on any tick". Every later, unrelated prime landing then stays owed.
- **Why it never clears:** `SettleRefused` re-reads the run as owned elsewhere on each tick and writes the record again. The new "unless this pass held a landing back" keeps it from being cleared. The only way forward named is to `lym adopt` the run back from its rightful owner.
- **Better outcome:** step 2 already handles an owned-elsewhere run by drop and hand-over (`HandOverOwnedElsewhere`) with no `done` write. Here the landing could likewise be removed and the run handed over, leaving the `done` write to its owner.

**2. BLOCKING — F4, card 11 step 1 and the `SettleRefused` paragraph.**
Changed text: "when that removes the last run `Dropped` names … clear the refused-push record step 2's rebase refusal wrote", plus "a rebase that removed the last commit `Dropped` names clears the drop's refusal through step 1 the same way".
- **The case:** a beam parent whose only landing was dropped. The drop removes that landing, so the beam pair target now holds only `Dropped` and no landings.
- **What breaks:** card 11 says `restartHubPushers` "runs `SettleRefused` for the landing target and for each pair target holding landings". A refused target starts no pusher, so neither a pass nor `SettleRefused` ever runs step 1 on that beam target.
- **Result:** the clearing rule never fires, and the beam parent stays refused after the rebase, which is F4's premise. The new `TestParentPush` row doesn't name a target, so it can pass on the landing target alone.

**3. MEDIUM — F1, card 6, the `PusherSync` paragraph.**
Changed text: "pushes the Bolt once through `Bolt.Push` with `OwedUnrecorded` and the single transport attempt card 5 gives the pusher's pushes".
- **The gap:** card 5 puts `SingleAttempt` on `fabriclayout.PairPush` only, and `PushPairAnchored` passes it on. `Bolt.Push(opts SyncOptions)` takes no `PairPush`, and no card adds a single-attempt field to `SyncOptions`. The current `SyncOptions` has none (`internal/fabricengine/fabric.go:74-95`).
- **Result:** as planned, the board push can't receive the single attempt. Its transport lookup would run the production backoff (`awaitOrigin`, `pushclassified.go:710-735`) inside the pusher, against D2's "no transport backoff of their own". The tier1 unreachable-origin row would also sleep through that backoff, against PATTERN-test-speed.

**4. LOW — F1, card 6 test row.**
Changed text: "with `BOARD_SKIP_PUSH` set, a committed but unpushed Bolt reads level".
- The setting is process-wide (`ApplySkipEnv` reads it with `os.Getenv`). The row has to use `t.Setenv`, which can't run in a parallel test.
- The row doesn't name that process-global state, which the testing rule requires when a test can't run in parallel.

F2, F5 and F6: I found no case where the proposal is wrong. The PR-landing rows in `finalize_pr_test.go` expect `fast-forward` and then `merge`, but that is the PR path, whose parent-side merge already finds the work in place. `NoFastForward` doesn't change it.

Tool calls:
- I grepped the landingshed and loomcli tests for fast-forward expectations, because my context didn't show whether a test pins a non-squash landing that fast-forwards (F2 breakage).
- In the same call I grepped card 11 for the `restartHubPushers` sentence, to confirm which targets `SettleRefused` runs for (F4 beam case).
