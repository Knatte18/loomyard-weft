```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the F8 proposal (build the range key from the full range diff, added files included) for a case where it is wrong or breaks something.

no findings

Cases I tried:
- **Wiring:** `driftDepsFrom` copies the whole `Deps` struct (`internal/shedrecipe/entries_drift.go:36-41`), so the new `RangeDiff` seam reaches the producer.
- **Records stored under the old key:** they mismatch once and are judged again, only for uncaused or carrying rulings.
- **Wider key:** any added file now triggers the re-judge. Before, any modified or deleted file already did, including production code, so the cost is of the same kind.
- **Merge-ins:** the diff starts at the review base, the merge base with the last merged parent tip, so the key covers the same range as the listed files.
- **Added carrier:** it is not in the judge's file list, but the judge's Step 2 searches the repository for carriers, so the re-judge can still see it.

Tool calls (1 of 6):
- One grep over `entries_drift.go` (`driftDepsFrom`), `ChangedFiles` users and `gitexec.Run`: my context did not show whether `driftDepsFrom` copies the struct or copies it field by field.
