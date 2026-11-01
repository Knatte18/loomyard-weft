```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each reviewer proposal the fixer carried out as proposed (F1, F4, F6, F7, F8, F9, F10) by building a case where it is wrong.

Proposals checked: F1 (start reading per platform), F4 (`Abandoned` set in `execution.writeRecord`), F6 (report leaves out `Abandoned` records), F7 (card-verify key by `card.ID()`), F8 (runner paragraph per verb), F9 (two lease-shortening writes), F10 (wall-time span).

- LOW, D5's added sentence "`lym gate report` leaves a record carrying `Abandoned` out of its run counts and its wait, wall and build totals." The proposal swaps one distortion for another. An abandoned runner holds its slots until its lease passes (D3 Bound: "at most the long cap (D5) plus the grace"). Once its records are dropped, the report no longer counts slot time that this machine spent. `runReport`'s doc (`internal/gatecli/report.go`, the comment above `runReport`) promises "the summed slot wait and wall time" of a key's records. With the new feature, cut runs become frequent: every lease pass and every supersede after a tree change. Dropping them hides that cost entirely, where the review only asked that cut tiers stop counting as outcomes. A narrower change keeps the cut tiers' wait and wall time in the totals and counts the cut runs apart from full runs, the way `buildRuns` and `carried` are already counted apart.

The other proposals hold. F1: on Windows `proc.CreationTime` returns false for a dead pid, so the join supersedes and never accepts a dead pid as live. F4: `reportBlocked` halts once the context is cancelled (`halt: e.ctx.Err() != nil`, `build.go`), so after a cancelled build only one marked blocked record is written and later tiers drop out with no record. F7: `runCardVerify` resolves `card` before it runs anything, so the key is available before the front locks. F8, F9 and F10 are wording changes, and I found no sibling that disagrees with them.

No further tool calls made.
