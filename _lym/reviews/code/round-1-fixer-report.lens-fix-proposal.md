```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged F1, F3, F4 and F6, and the other proposals carried out as proposed. I found nothing to report.

- **F1 (restore status and child-recipe records on a failed removal):** I built the retry cases and none broke.
  - The restore leaves the seed, settle records and notice episode deleted on disk while the records HEAD still tracks them.
  - A re-run of `lym beam create`, or of `lym batten cleanup --apply`, still finishes the removal. `judgeRunDirectory` and `clearMotifRun` read only the restored done status.
  - The retry's `removeRunRecords` commit takes the whole run directory as its pathspec, so it also commits the deletions of those unrestored files.
  - The fixer restored only what the next judgement reads. A full restore would add a copy step for no gain.
  - If the records commit lands but the call still returns an error, the restore writes a status file the HEAD no longer tracks. The retry then finds the run untracked, removes it, and makes no commit. This case is unlikely, since `CommitAnchoredPaths` is a no-push commit, and it breaks nothing.
- **F3 (new `clearMotifRun` rows):** the untracked row is sound. `runRecordsTracked` checks the status path and the seed path, and the table writes neither into `stubs.committed`. The scratch-block row restores the directory mode in a `t.Cleanup` registered after the `t.TempDir` cleanup, so the permission change is undone before the temp directory is removed.
- **F4 (overview text):** the claim that any other done run is still refused, naming `lym batten cleanup --apply`, holds against `internal/battencli/arm.go:880`.
- **F6 (comment rewording):** the new `clearMotifRun` doc is self-contained, and the dropped `ReadChildRecipe` sentence leaves no stale reference.

No finding rises to LOW or higher.

Tool calls made:
1. `grep` for "batten cleanup --apply" in non-test Go files, plus the `CommitAnchoredPaths` definition. My context lacked the text of the refusal that F4's overview sentence describes, and the commit semantics behind the restore.
