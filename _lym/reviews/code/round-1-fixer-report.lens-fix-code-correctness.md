```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

Each defect the review names breaks the new row:
- **Error branch flipped to `false`:** with `programNameErr` set and an unnamed caller, `noticeFits` returns false at `decision.go:667`. `decisionVerb` then refuses, and the row's `code != 0` check fails.
- **Eager lookup restored:** the resolver error would abort `decision add` before the append, and the row fails the same way.

Traced step by step:
- **`programName` returns `("hub:LOOM", f.programNameErr)`.** The name is non-empty, but `noticeFits` ignores it once `err != nil`, so it returns true.
- **Row `{by: "parent", programNameFails: true}`.** It has no `strandName`, so the strand check in `noticeFits` is skipped, and the notice line is short. The verb appends, commits once and exits 0. The row's assertions (`commits == 1`, prior record prefix plus the entry, a UTC date) hold.
- **No live strands.** `liveStrands` is nil, so `notifyLiveRuns` posts nothing and `programName` is not called again. The row doesn't touch the notice-posting path.
- **Other rows.** `programNameErr` defaults to nil, so every other row behaves as before. The `deps()` change is the only edit to shared test code.
- **Stray edits.** The first perl pass touched only the field and the `deps()` line. The loop edit went in through Edit, and the diff shows no duplicate `programNameFails` field.

The row doesn't exercise the multi-entry branch at `decision.go:344`, which also calls `noticeFits`. The reviewer's `fix` key asked only for the one-entry case, so this isn't a finding.

Tool calls (1):
- Read `decision.go:655-680` to confirm the per-entry `noticeFits` call site and that a false return refuses; the context only showed `noticeFits` itself.
