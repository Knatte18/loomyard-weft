```yaml
lens: fix-way-forward
outcome: reported
```

Task: check that the ways forward added by the F12 fix in `_lym/discussion/decision-record.md` D4 step 2 can be carried out and lead to progress.

Ways forward the fix added:
1. The local read of `_lym/landing/beam-ready.json` in the beam's worktree when that worktree is on the orch's machine.
2. For any other beam, `git -C <records repo> fetch origin`, then `git -C <records repo> show origin/<beam branch>-weft:<anchor>_lym/landing/beam-ready.json`.
3. When no file is found, the orch asks the beam's orch with `lym creel post --expect-reply`, or asks the operator, and the beam qualifies only on a "no".
4. A found file, or a "yes", fails the check. The existing "When the check fails" line then lifts the pause and tells the operator.

Findings:

- LOW, D4 step 2, "An absent file proves nothing ... the orch asks that beam's orch by `lym creel post --expect-reply`, or the operator". The text gives no point at which the orch stops waiting for that beam's orch and asks the operator instead. Every other wait in D4 step 3 has a bound, such as the ten-minute teardown wait. Evidence: the added sentence is the only unbounded wait among D4's checks. Nothing is lost, because the slug pause stays held and the move stops before the cancel.
- LOW, D4 step 2, `git -C <records repo> show origin/<beam branch>-weft:...`. When the beam's records branch was never pushed, `origin/<beam branch>-weft` does not exist and `git show` fails with "invalid object name", not "path does not exist". The text covers only "A file found there" and "An absent file", so a reader may take the error as a failure of the procedure rather than as the same outcome as an absent file. Evidence: the added lines name only those two outcomes. The commit in `MarkReady` → `CommitAnchoredPaths` makes no push, as `internal/fabricengine/commitweftpaths.go` shows.

Tool calls:
- One grep of `RecordsBranchName` in `internal/fabricengine/branchname.go` and of the records push sites, to confirm the `-weft` branch name and whether records branches reach origin. My context did not include those files.
