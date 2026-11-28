```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of the change set (agreement with siblings and twins).

1. **MEDIUM**: `_lym/landing/summary.md`, "What changes", the `SlotWaitSeconds` bullet (left as it was).
   - It says `lym gate report` uses the helper "instead of computing the same figure inline, so the report and the summary line always agree."
   - It is no longer the same figure. The old inline formula, and the helper before this fix, were `Start − WaitStart`. With `slots_held` set, the report's `wait_seconds` and `abandoned_wait_seconds` now leave out the shared build and the `--copies` prebuild.
   - That change to the report's totals is stated nowhere.
   - "Always agree" also sits next to the new sentence that a rerun's line shows a wait its own record does not hold. Only the totals agree.

2. **LOW**: `_lym/landing/summary.md`, Design decisions ("The build's time is on the build line and not in the tier's wait"), with `internal/gatecli/doc.go` ("Work done inside the held slots before the tier's command…") and the help at `internal/gatecli/cli.go:85`.
   - The `--copies` lym prebuild has left `wait` and is not in `wall` either: each copy's `started` is taken after the prebuild, and `mergeCopies` takes the longest copy's `WallSec`. It now appears on no line.
   - A run without copies counts the same prebuild in `wall`.
   - The summary accounts only for the shared build's time. The help's "run time without that wait as wall" does not say the copies' prebuild is left out.

3. **LOW**: `_lym/landing/summary.md`, title "Show each tier's slot wait beside its wall time".
   - The field now stands before `discovered`, with the counts between it and `wall`, so "beside" no longer describes the line.

No other twin was left stale in what I searched. The `gate report` help (`report.go:35`), `doc.go:672`, `callSpan` and the reader patterns still agree with the change.

Tool calls:
- One `git grep` for slot-wait and summary-line mentions across Go, Markdown and YAML, plus `cat _lym/landing/summary.md`: my context held neither the repo-wide mentions nor the summary as it reads after the edits.
