---
round: 1
focus:
  - "Commit 382a591f (`internal/heddleengine/overview.go`, `overview_test.go`) follows the merge. The amended board entry allows only a change the merge needs, with no refactor and no fix of an older fault. Settle whether the merge breaks anything this commit repairs, or whether the commit is a change the entry does not license."
  - "`_lym/landing/summary.md` says the merge is `bae0ec63` 'with no commit after it'. The range holds commit 382a591f after the merge. Settle whether the summary's claim is borne out by the range."
  - "The three conflict resolutions in `bae0ec63`, read through `git show --remerge-diff bae0ec63`: `internal/beamcli/pr_test.go`, `internal/lymdirs/doc.go` and `internal/reedengine/bindings_integration_test.go`. Settle whether each keeps what both sides meant and leaves no dropped assertion, unused field or stale reference. In `pr_test.go`, check that the `wantNot` field and its check are gone everywhere and that the handed-over row still asserts that its refusal does not say 'again'."
  - "Test economy in the `pr_test.go` resolution: settle whether the merged `TestPROpen` loop applies `wantNoErr` and `wantUntouched` to the rows that set them without leaving a row that silently asserts less than before."
---

The review range is the merge's conflict resolution plus the one commit after it; the parent's own commits are no subject.
Checks done while seeding: `recorderDeadline` is 30 seconds, as the resolution of `bindings_integration_test.go` claims.
`HandOverDirName` and `WeaverDirName` both exist in `internal/lymdirs/dirs.go`.
