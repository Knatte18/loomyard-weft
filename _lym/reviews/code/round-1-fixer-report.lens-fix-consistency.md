```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix commits `3788581b` and `685f2ce7` for agreement with their surroundings. Two findings, both LOW.

1. **LOW** — `internal/beamcli/dobbypairanchor_test.go`, `TestDobbyRuns_ReadsTheLandOutcomeAndTheLandingCommit`.
   - **Summary:** The test now also pins `EntryEnded` and `PairPresent`, but its name no longer says so.
   - **Evidence:** The `landing` struct and the `want` rows now assert `entryEnded` and `pairPresent`. The "pairless" fixture and the board.json write exist only for those fields.
   - **Reach:** The name says only the land outcome and landing commit. The next reader looking for where `EntryEnded` or `PairPresent` is pinned would not find it from the name.
   - **Note:** The review said to add no new test function, so a rename is the only way to bring the name back in line.

2. **LOW** — `internal/beamcli/dobbypairanchor_test.go`, file header, line 2.
   - **Summary:** The header says the file pins what `DobbyRuns` reads of "a run's place against its Land row". It now also pins the ended-entry and pair-presence reads.
   - **Evidence:** The header is the file's disposition contract (code-quality's file-header rule). The diff adds board-entry and pair-path reads to the file's contract.

Everything else agrees:
- **`ended_test.go` header:** It still covers the new rows.
- **`ended.go` doc:** The 13-failure ceiling row matches the doubling it documents (10s doubling, capped at `endedTeardownCeiling`).
- **`loop.go`:** The nil-seam row and the binary-check row match the `d.teardown == nil` guard at `loop.go:564` and the early return in `tearDownEnded`.
- **Test-local `err`:** The `pairless, err :=` declaration is the only `err` before `views, err :=`, which declares `views` fresh, so there is no redeclaration conflict.

Further tool calls: 2 of 6. One `git diff HEAD~2 HEAD --stat` with the beamcli diff, to see the formatter's final form of `dobbypairanchor_test.go`, which my context lacked. One `grep` of the file header lines, to check the header text.
