```yaml
lens: fix-consistency
outcome: reported
```

No findings.

- **F2 doc line.** The new `TestScheduler_HookWake` comment line says the bell is held within 30 seconds of a ring while no turn end was read after it, since the woken turn's Stop hands out the mail. This matches `wakeHookStrand` at `deliver.go:287-290` and its doc comment at `deliver.go:256-257`. It uses the same "30" wording as the line above it.
- **F1 row.** The new row has the same shape as its two sibling rows. It takes `ringAt` before `ackedRing`, posts "two" under `settle`, and asserts `untilBell` is at least 30 seconds. A zero `TurnEndedAt` makes `woken` true at `deliver.go:287`, because `Time.After` is false for a zero time. The probe that rings still passes `ringable` at `deliver.go:281`, because the zero time satisfies `idleness.TurnEndedAt.IsZero()`, so `ackedRing` works unchanged. The row sits after the two new rows and before `AWaiterThatEndsWithoutARingStartsTheGraceAgainFromItsEnd`, as F1 asked.
- **Twins.** The change adds no test twin that needs a matching edit, and `doc.go` and `deliver.go` were not touched.

Tool calls: none; the context already held every file and the review.
