```yaml
lens: fix-consistency
outcome: reported
```

**Task:** check the fix-consistency lens over the change set: siblings and twins of the five fixes.

**Findings**

1. **BLOCKING.** `internal/creelengine/deliver.go`, the `!strand.PaneMail` branch in `probe` (now `return false, len(snap.injects) > 0`).
   - **Summary:** The F2 fix turns a slow poll into a tight spin for a strand with unread mail and no mark.
   - **Evidence:**
     - `run()` reacts to `pending == false` by calling `w.retire()` and then `continue`, with no wait.
     - `retire()` returns false whenever `store.hasPending(receiver) && !stopping`. Unread mail counts as pending, as `hasPending` is the store's pending check.
     - So for a strand with neither `CreelHooks` nor `PaneMail`, unread mail and no queued inject, `probe` returns `(false, false)`, `retire` returns false, and the loop re-probes immediately.
     - That repeats with no `clock.After` or `wake` wait, and it also takes `paneLock` and `snapshot` on every pass.
     - Before the fix, `(false, true)` reached the backoff wait at `run()`, capped at `probeBackoffCeiling`.
   - **Note:** The review's own `fix` text said to return `false, false`. It has the same flaw, because the worker loop treats "not pending" as "the store has nothing", not as "nothing deliverable".
   - **Possible resolution:** Keep returning pending true and let the backoff run, or have `retire` agree with `probe`.

2. **No findings** for the other twins I checked.
   - `tools/sandbox/SANDBOX-CREEL-SUITE.md:115` now matches `MessageLine`'s `from "<sender>"`.
   - The only other unquoted shape is `docs/research/creel-hook-delivery-2026-10-25/report.md:32`. It is a dated research record and the generic `<sender>` form, so I don't count it.
   - `programName` is read only at `decision.go:307`, and the test fake at `decision_test.go:140` was updated to the new function type.
   - `doc.go:245` ("one worker per receiver with a queued inject or unread mail") is still accurate in wording. Findings 1 and 2 concern how the fixed worker behaves, not that line.

3. **LOW.** `internal/creelengine/doc.go:245`, with `run()`'s comment ("the worker ends when nothing is pending").
   - **Summary:** No doc or test records the new F2 behaviour, a mark-less strand ending its worker.
   - **Evidence:** Neither `doc.go` nor `probe`'s doc comment (`deliver.go:197-199`) mentions it. This depends on how finding 1 is resolved.

**Tool calls**
1. `grep` for the old and new sender shape and for `PaneMail` prose in `.md` and `doc.go` files: my context lacked a repo-wide search for twins.
2. `grep` over `internal/creelengine/doc.go` for probe, worker and pending prose, and over `tools/sandbox/*.md` for `EXTERNAL`: my context lacked those doc lines.
3. `grep` for `programName` and `FormatProgram` across `internal/loomcli` non-test files: my context lacked a full usage list.
