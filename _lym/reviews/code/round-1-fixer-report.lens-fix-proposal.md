```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals that the fixer carried out as proposed (F1, F2, F3, F5, F6, F10, F11, F12, F13 and the test-addition fixes), looking for a concrete case where a proposal is wrong or breaks something.

Proposals reviewed, none disputed by the fixer. The F7 lock change was the fixer's own, outside the proposal.

- **F2:** I tried a fork with run records tracked but absent from disk, and a fork whose `_lym` is wholly absent. In both, `moveForkedRecordsToAnchor` appends the source spec and skips the rename. `entryMatchesWeft` matches it through `--cached`, so the deletion is committed. A move to anchor `.` skips `MkdirAll`. A fork with untracked leftovers under `_lym` cannot occur, because the fork is no-checkout. The new row exercises the run-records case, since the earlier row had committed run records into the prime. No breakage found.
- **F3 and F1:** the PATTERN header line ("bar a prime push failing `protected`; an `operator_unpushed` push is owed") agrees with the revised body. The README text mirrors `docs/getting-started.md` and leaves out its third hazard (direct pushes refused). F1's fix key asks only for the switch, reset and stash hazards plus `--single-repo`, so this is not a finding.
- **F11:** moving HEAD back with `update-ref HEAD <head> <sha>` leaves MERGE_HEAD and the index intact, so the merge state is unchanged. The unsettled path's later `resetMergeSides` failure is not covered by the undo, but F11 did not name it.
- **F12 and F13:** nothing breaks. `git init` in the prime makes `oneRepositoryAt` log nothing (no commits, no `lym-bolt` branch), and the gate passed.
- **F10:** the 300 ms wait is real time, but it can only fail in the direction of a real defect, because a missing retry wrapper makes `done` fire early.
- **F5:** the fixer's own change, parking through `MergeIn` on the receiving side, is the only way a merge stays parked (the first attempt showed `Merge` aborts on conflict). It matches the guard, which reads the receiving side's merge record.

No findings.

Tool calls: none. My context already held every changed file and the gate results.
