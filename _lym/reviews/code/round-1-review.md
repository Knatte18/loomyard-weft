---
verdict: APPROVED
---

The diff adds tests only, and it matches the amended entry as the change description claims.
The two ownerless-beam tests, the two decoy rows and the teardown test each fail if their behavior breaks, and none duplicates an existing test.
The entry's other pins already exist: the hold's skipped steps in `TestRun_ForeignOwner` and `TestRun_AnAdoptHoldSchedulesNothingUntilItLifts`,
the tip decision rows in `TestArchiveOwnerTips`, and the pairs end to end in `TestArchiveOwnerTips_CoversTheWorktreesOwnPairAndTheRunsUnderIt`.
No commit carries a `Pattern-Departure:` trailer, and none is needed.

## Rejected

- Lens code-tests, LOW scope: the records-tag-versus-branch rows pin `ReadOwnerArchives`, not a `lym adopt --here` path.
  Rejected because the records-tag-against-records-branch judgment lives only in the `--archives` read (`internal/adoptcli/doc.go:135`), and no `--here` code reads a records tag, so the rows pin the one place the two differ.
