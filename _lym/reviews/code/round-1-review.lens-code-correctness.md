```yaml
lens: code-correctness
outcome: reported
```

Task: review the code-correctness of the diff by running concrete cases through each changed function and error path.

**Per function or branch: the case I ran, and whether it is correct**

- `UpsertTaskWithRecord`, run-agent guard: a strand name that parses as a run agent is refused before any read or write. Correct.
- `UpsertTaskWithRecord`, empty or missing slug: `judgeRecordReplaceIn` returns the "no board entry" refusal and nothing is written. Correct.
- `judgeRecordReplaceIn`, refusal order: absent, then no record, then done, then run-held, then worktree (skipped for a filed family project), then live weft, then probe failure. This matches D4. Correct.
- `judgeRecordReplaceIn`, family exemption: a project whose slug equals `filed.Project` in the raddle store is exempt from both the worktree and the weft refusals. The run-held check comes before it, so the run lock still applies. Correct.
- `UpsertTaskWithRecord`, critical section: the record is written after `UpsertTask` and before `Save`. The stamp is computed from the post-upsert body. Correct.
- `UpsertTaskWithRecord`, failure after the record write: `recordWritten && !saved` restores `former`. A failure in `Save`, `checkRunLocks`, `checkProjectRules`, `checkDiscussedPromotion` or `checkTaskCreation` is covered. The rewrite of `former` is correct, but its error is discarded.
- `UpsertTaskWithRecord`, failure after the save (render or `followDecisionRecords`): `saved` is true, so the record is kept and the error is returned. The saved write is reported as an error. This is acceptable.
- `StoreDecisionRecord`: an entry flagged discussed with no stamp is not skipped as a no-op, so the first re-store adds the stamp. A matching stamp, flag and bytes is a no-op. Correct.
- `recordBodyChanged` and `PromoteTo`: no stamp gives no warning, a mismatched stamp warns, and the body is read before `promoteTo`. A refused promote returns an empty result. Correct.
- `fileRaddleEntry` and `convertRunEntry`: the stamp is taken after the body is final. Correct.
- `CheckRecordSections` and `Validate`: the finding order and the `Path` stamping are the same as before. A nil record yields a missing-section finding for each required section. Correct.
- `readRecordFile`: an unreadable path and a failing record both return an error with a way forward, and nothing is written. Correct for a file path.
- `weftLiveness` in `Add`: with `probesOrigins()` false it returns `{live: localExists}` with a nil error, which is what the old default gave. With it true it calls `probeWeftLeftover`. Correct, and behavior is preserved.
- `RecordsBranchLive`, local branch present: it returns true with no origin read. Correct.
- `RecordsBranchLive`, no local branch: it calls `probeWeftLeftover(false)`. An absent origin branch gives not live, a covered archive tag gives not live, anything else gives live, and an error returns `(false, err)`. Correct.
- `recordsBranchLive` in `cli.go`: a nil `location`, which is the `--board-path` case, returns an error. The refusal text is wrong for that case (F2).
- `replayFields` (the replay merge table): `record_body_sha256` is missing from it (F1).

**Findings**

- **F1 — MEDIUM, design.** `internal/boardengine/replay.go:190-217`. `record_body_sha256` is not in `replayFields`, which lists the Task fields a replay merges.
  - D5 says a board sync's replay carries the stamp with the entry. The new key is now a known field, so it is not in the "unknown keys stay as origin holds it" group either.
  - When another machine runs `upsert --record-file`, the body and the record file replay but the stamp does not. Origin then keeps the old stamp against the new body, and a later promote gives a false "body changed" warning.
  - Fix: add a `record_body_sha256` row to `replayFields` and a replay test. I did not check whether the merge for a new entry copies the whole entry, which would carry the stamp in that case.
- **F2 — LOW, design.** `internal/boardcli/cli.go:73-79` with `internal/boardengine/recordupsert.go:36-38`.
  - A nil `location` or a `LoadConfig` failure surfaces from `weftLive`. `UpsertTaskWithRecord` wraps every probe error as "could not be probed … way forward: re-run the upsert once origin answers".
  - That way forward is wrong for `--board-path` or a config error, where re-running never helps. Under `--board-path` this probe error also replaces the write's own refusal "board outputs not configured".
  - Fix: make the way forward in `UpsertTaskWithRecord` neutral, or return a distinct error from the closure.
- **F3 — LOW, design.** `internal/boardcli/recordfile.go:167`.
  - The flag is meant to mirror `--body-file`, which reads stdin for `-`. `readRecordFile` calls `os.ReadFile(path)`, so `-` names a file called `-`.
  - The flag's help text does not promise stdin, so this is a mismatch with the mirror claim only.
  - Fix: either accept `-` or state in the help that stdin is not read.

**Tool calls**

1. `grep` of `RecordBodySHA256` and the known-key handling across boardengine, to see whether other readers or writers of the field exist. My context lacked this.
2. `sed` of `replay.go` 150-260, to check how replay carries entry fields. The diff does not touch it.
3. `grep` of `replayFields` and `Table[Task]` in tests, to see whether any test covers the stamp. None does.
