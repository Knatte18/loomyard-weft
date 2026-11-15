```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran the search behind each finding for sets the fix may not have covered.

**BLOCKING: F1 (`internal/runfsm/act.go:186`) breaks `lym batten cancel` of an already-cancelled run.**
- `Aside.Cancel` is also used by `cancelAside` at `internal/battencli/cancelcmd.go:460`, which `cancelVerdict` passes to `Judge`. The cancel verb's `Verdict` seam calls it at `cancelcmd.go:294`. The review named only the `resumeAside` caller.
- Before the fix, `cancelAside` dropped a finished cancel intent, so a cancelled run read `running` and the cancel verb's act rule passed.
- After the fix, a finished cancel stays. The run reads `Cancelled`, `Act` answers wait/`ReasonTerminal`, and `verdict.Refusal` at `internal/battencli/cancel.go:335` returns `terminalRefusal`.
- That refusal blocks the cancel path the verb offers for a run already cancelled with its pair kept:
  - `cancel.go:339-341` returns "already cancelled with its pair kept; way forward: run `lym batten cancel <slug> --remove`". It sits after the verdict check, so it is now unreachable.
  - `act.go:154` (`terminalRefusal`) names `lym batten cancel <slug> --remove` as the way forward for a cancelled run, and that command is now refused too.
- `lym batten cancel --remove` over a cancelled-and-kept run, the documented way to remove it, now fails with a refusal that tells the operator to run the same command.
- The gate passed, so no test drives `cancelVerdict` over a finished cancel through the real `Judge`.
- Proposed fix: keep the new rule for the resume path only, for example a separate flag such as `Aside.UnfinishedCancel`, with `resumeAside` using it and `cancelAside` keeping `Cancel`. Add a test that `lym batten cancel --remove` passes the verdict over a finished keep-pair cancel.

**Other sets, no findings:**
- F2 (callers of `cancelMarkerRefusal`): the callers at `startguard.go:289`, `startguard.go:489`, `cancel.go:355` and `cancelmarker.go:156` all go through the one function, so the new wording covers them. The `takeCancelMarker` caller at `cancelmarker.go:154` discards the read error and builds a reading with an empty intent when it fails. If the record is torn, it will print the "does not decode" wording even though a live writer holds the lock. `WriterAlive` is checked first in `cancelMarkerRefusal`, so that case still gets the live-cancel wording. No problem.
- F4 (contained tests): the only source I found for the contained list is `round-1-focus.md:14`, which names exactly the three tests that were marked. The report itself isn't on disk under `_lym`, so I could not rerun its full list. Left unchecked.
- F3 (resume of a run another machine owns): its premise is a single missing test, not a set.

Tool calls made:
1. `grep` for `Aside{` and `Cancel: true` users, `cancelMarkerRefusal` and `readCancelMarker` callers, and files mentioning "contained". Reason: the review named only one caller per set.
2. `grep` for `cancelVerdict` and `terminal` handling in `internal/battencli`. Reason: to see how `cancelAside` is consumed.
3. `sed` of `cancelcmd.go:280-300` and `cancel.go:325-360`. Reason: to confirm the verdict check runs before the "already cancelled" branch.
4. `sed` of `cancelcmd.go:450-475` plus a `grep` of `round-1-focus.md` for "contained". Reason: to read the `cancelAside` definition and find the contained-test list.
