```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the review's proposals that the fix carried out as proposed (F1, F2, F3, F5, F6, F7, F9).

no findings

I built the likely failure cases from context, none of which holds against the changed code:

- **F1, F3, F5 (cancel counts an unfinished teardown):**
  - A running child's record is not listed, so it never counts as left.
  - A failed step keeps `Rerun` in the record, so the cancel it names still matches.
  - A discard record that names the cancel is settled by the same path.
  - A failed origin lookup returns an error rather than `ErrPairNotFound`, so no record is dropped while origin is unread.
  - With part of the pair left, `SettleGonePairTeardown` keeps the record.
- **F3's kept condition-drop:** the cancel now does finish in the state where origin holds nothing, so the reviewer's "unless" branch applies.
- **F7:** the test holds the lock exclusive rather than shared. It still reaches the `lockRerun` branch.
- **F2, F6, F9:** the changed text agrees with the code as changed.

Noted outside this lens, as it predates the fix and I did not verify it by running anything: there is one pending record per slug. A cancel's later discard can therefore overwrite a failed removal's record.

Tool calls: none.
