```yaml
lens: fix-claims
outcome: reported
```

Task: check that the code claims in the fix hold against the code at HEAD.

**1. MEDIUM: D2's second bullet relies on `pauserecord.Admit` for something it does not do.**
- **Changed text:** "A transition's push passes the pause record's network gate, as `pauserecord.Admit` lets a lock holder through, so a pause or hold written after the commit stops the run's next producer and never that push."
- **Summary:** `Admit` waits at the gate before it admits anything. A push that takes its admission after the commit therefore still waits out a pause written between the commit and the push.
- **Evidence:** `internal/pauserecord/hold.go:463-464` makes `Admit` call `Network(op)` first, which waits while a pause covers the process. Its doc says "admitted before it takes the lock, the holder is never held". The claim holds only if the run takes the admission before the commit and keeps it through the push. D2 never says that.

**2. MEDIUM: the D5 refusal's way forward names a flag that does not exist.**
- **Changed text:** "its way forward is `lym machine name --forget-former <name>` on this machine", together with the new D5 criterion.
- **Summary:** nothing in `internal/machinecli` has a forget flag. The record also never marks the flag as new, as it does for `internal/runfsm` (new), so a plan writer reads it as an existing step.
- **Evidence:** `grep -rn forget internal/machinecli/` returns nothing.

The other claims I checked hold:
- `battencli.ClearDoneRun` deletes the task branch on origin (`discardTaskBranchIfPresent`, `cleanup.go:152-178`), and it clears only a status that `judgeStatus` judges done (`cleanup.go:222`).
- `pairteardown` `Request.Local` takes no origin step and writes no board (`teardown.go:54-56` and `:419`).

Tool calls:
- One grep of `internal/machinecli` for a forget flag, plus `clearDoneRun`'s body: my context lacked both the machinecli flags and the body of `clearDoneRun`.
