```yaml
lens: fix-proposal
outcome: reported
```

Task: fix-proposal lens, challenging the reviewer's proposals the fixer carried out (F1, F2, F4, F5, F6).

Challenged: F1, F2, F4, F5, F6. F4, F5 and F6: no breaking case found.

1. **LOW** (F1). Changed text: `RunRecord.SlotWaitSeconds` and the `slots_held` stamp in `runWithSlots`.
   - **Summary:** under `--copies`, the time of the `lym` prebuild now appears in neither `wait` nor `wall`, on the line or in `lym gate report`.
   - **Evidence:** `runTierCopies` (`internal/gatecli/copies.go:64-79`) prebuilds before any copy's `runTierCommands` takes `started`, and the merged `WallSec` is the longest copy's own wall. The old `Start - WaitStart` counted the prebuild as wait; ending the wait at `SlotsHeld` drops it.
   - **Contrast:** the shared build has its own line. A run without copies counts the same prebuild in `wall`, because `runInvocations` runs after `started`. This prebuild has no line.
   - **Case:** a `--copies` call on a `prebuilt_lym` tier with a 60 s cold build shows `wait 0.0s wall <copy time>`, and the 60 s is not reported.

2. **LOW** (F1/F2). Changed text: `_lym/landing/summary.md`, "What changes".
   - **Summary:** the new rerun sentence sits beside an untouched claim that it partly contradicts.
   - **Evidence:** the fix wrote "A rerun's line shows the slot wait the call spent, while its record holds none". The next bullet still ends "`lym gate report` now uses it ... so the report and the summary line always agree."
   - **Case:** on a rerun, the record named by the line's own `log` path gives a slot wait of 0 while the line shows the call's wait. Only the report's totals agree with the lines.

Tool calls:
- One `git grep` for text describing the slot wait (`wait_start`, `wait_seconds`, "slot wait"): my context did not show which other docs or help describe it. Nothing in the matching lines contradicts the change.
