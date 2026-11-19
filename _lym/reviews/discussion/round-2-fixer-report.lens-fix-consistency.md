```yaml
lens: fix-consistency
outcome: reported
```

Task: check whether the F14 change set agrees with the sibling text in the decision record and support log.

- LOW — Goal, `decision-record.md` line 7: "a records store wired exactly as a two-repository hub's weft".
  The changed D1 now opens "in every part but the board's branch".
  The changed D3 also builds the `--single-repo` store by `git init` "never by the two-repository weft clone", and hands `ensureBoardWorktree` the board name rather than the code branch.
  "Exactly" in the Goal is the twin of D1's old "unchanged", and the fix left it as it was.

- LOW — D3 Rationale: "seeding through the existing weft bind keeps one seeding path".
  The changed `--single-repo` paragraph now departs from the weft bind in three places: how the store is made (`git init` and a fetch), the board's branch, and the new `lym.primeRecordsBranch` write.
  The rationale still claims the bind is reused unchanged and doesn't name these departures, though the paragraph's "except that…" covers only the board branch.

- LOW — `support-log.md` Rejected alternatives, "Records branches in the code store itself": "two stores keep the local geometry identical".
  That contradicts D1's new "in every part but the board's branch".
  It is the same stale twin of the removed "unchanged" claim.

- LOW — D5, in the bullet "the git config key `lym.primeBranch` (removed)", against D1's new `lym.primeRecordsBranch`.
  D5 removes one `lym.prime*` config key while D1 adds a near-namesake in the other store, and neither text separates them.
  The support log's new rejected-alternative entry separates the new key from the layout refspec, not from the removed `lym.primeBranch`.
  A plan writer reverting f06acf39 by D5's list could take the new key for the removed one.

No further tool calls made.
