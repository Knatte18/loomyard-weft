```yaml
lens: fix-proposal
outcome: reported
```

No findings.

I checked F1's proposal (add a decision-add table row whose `programName` returns an error, then assert a one-entry add is recorded and `noticeFits` does not refuse) against `internal/loomcli/decision.go:666-670`.

- **Reverting to an eager lookup:** the added row would fail, because the error would abort the add before the append. The row has `strandName` empty, so `noticeFits` skips the strand check and does call `programName`.
- **Flipping the error branch to `false`:** the row would also fail. The check at `decision.go:667` runs for every entry, including a single one, with no `len > 1` guard. A `false` there makes the add refuse with `decisionWayNoticeTooLong`, and the row expects exit 0, one commit and the entry in the record.
- **The fake:** `programNameErr` in `decisionFake` defaults to nil, so every other row keeps its previous behaviour. I found no sibling that needs the same change.
- **Weaker spot:** the second `noticeFits` call, in `notifyLiveRuns` (`decision.go:344`, only for several entries), is not pinned by the new row. The new row has no live strands and a single entry. F1 did not ask for it, so this is not a finding.

Tool calls: 1. I read `internal/loomcli/decision.go:655-679`, because my context had only the grep hit for line 667 and not whether the call there depended on entry count.
