```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the change set through the code-correctness lens.

**Findings**

- **LOW** — `destroy.go`, the `fail` closure in `pushCloneSeed` (the new `else` branch).
  - **Summary:** The appended "the records branch was returned to what origin held before" lands after the way forward.
  - **Evidence:** `seedPushCause` builds "…; way forward: run the same clone again", and `fail` then appends the clause after it.
  - **Effect:** The message ends in a status clause instead of the way forward. The existing "also failed" clause already sits there, so this is consistent but slightly awkward. The reviewer's F5 fix proposed exactly this placement.

No other findings. I ran each changed branch with one input:

- **`refuseUnseededBoard`, unborn board:** `CurrentSHA` returns `ErrNoCommits` and the refusal names a join. The "while this board holds no commits" row matches.
- **`refuseUnseededBoard`, unrelated origin board:** `sharesHistory` gets exit 1 from `merge-base` and returns false. That gives the "shares no history" refusal, and the unrelated-board row matches.
- **`refuseUnseededBoard`, origin board descending from the local one:** `Fetch` brings the commit into the store and `merge-base` succeeds. That names the upstream settings, and the descendant row fails against the old `isAncestorOrEqual`, which answered false for a missing commit.
- **`refuseUnseededBoard`, failures of the calls it makes:** a failing `Fetch`, `CurrentSHA`, `merge-base` (exit 128 on a missing object) or config read each takes a wrapped path with a rerun way forward.
- **`pushCloneSeed`, board moved by another clone:** the ref listing returns a tip that is neither the pushed nor the leased one, so `withdrawUnansweredSeedPush` returns `(false, nil)`. The error wraps `ErrLeaseRejected` with `%v` on the original, so `seedRefusedByOrigin` is false and `seedPushCause` takes the join case. Origin's board is left alone and the records branch is restored.
- **Callers and tests:** `killPushClientHook` has one caller, which is updated. The `BoardLanded` fragments match the rewritten messages. `isAncestorOrEqual` is still used by `resetUnnamedTip`, so removing its use here breaks no caller.

The gate log shows `BoardMovedToAnotherTipBeforeTheKill` ran in tier 2 on Linux, and the log shows its sibling killed-push rows passing. I did not read its pass line. The tier 1 rows passed in the gate run the parent made, according to the parent's gate output. The "origin cannot be read" and "committed ancestor" rows are coverage rows, so they have no defect to fail against.

**Calls made**
1. `grep SeedPushKilled` in the tier2 gate log: to confirm the new killed-push row ran and was not skipped, since the context only showed "skipped 3".
