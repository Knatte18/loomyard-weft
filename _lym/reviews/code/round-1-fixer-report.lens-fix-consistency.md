```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency found two LOW inconsistencies in the F1/F2 change set. Neither is a contradiction, and I found no stale twin that would ship a wrong behaviour.

**LOW — `loomCommitStatusDeps` doc, `internal/loomcli/wiring.go:194`.** The text now disagrees with `statusCommitPathspec`.
- It still reads "scoped to statusCommitPathspec (the status file, plus the review round record, the loom durable directory and the drive reports when each holds a file)".
- That list already omitted the arbiter and tackler directories, and now omits the settle records too.
- The edit updated the `statusCommitPathspec` doc at line 85 but left this caller-side summary as it was.
- Suggested fix: reword it to point at `statusCommitPathspec` without enumerating the contents.

**LOW — `internal/preflight/preflight.go` comment, F1 wording.** It says "The status commit stages the settle records and the attempt directories where it can".
- This holds for `loomcli`'s `statusCommitPathspec` (settle records, `arbiter/`, `tackler/`) and for the driver's `CommitRecords`, which reuses it.
- It does not hold for `battencli`'s `battenRunCommitPaths`. That function now stages the settle records, but not the arbiter or tackler attempt directories. The review's own `fix` text asked for the settle records only.
- The sentence is therefore true for the loom path and only partly true for the batten path, which the "where it can" hedge covers loosely.
- Suggested fix: reword to "where it stages them", as the review's `fix` text proposed.

**Twins checked and found consistent:**
- **`shedrun` functions.** `SettleRecordsRel` sits beside `SeedRel` and `StatusRel` and has the same shape. Its `paths_test.go` row sits with its peers.
- **Test coverage.** The two pathspec tests (`wiring_commitstatus_test.go`, `commitstatus_test.go`) cover the new entry, and the `battencli` one appends in the same order as the code.
- **Other `SeedRel`/`StatusRel` callers.**
  - `battencli/cleanup.go:261` deletes seed and status.
  - `battencli/wire.go:1426` commits a child seed.
  - `loomcli/sharedbootstrap.go:236` is the bootstrap commit.
  - None is a status-transition pathspec, so none needed the settle records.
- **`shedrun/doc.go:37`.** It says the settle records sit beside the drive reports and makes no commit claim.

Tool calls, 2 of 6:
- **First `git grep`:** for `DriveReportsRel`, `NoticeEpisodeRel` and `SeedRel` callers outside tests, to find sibling pathspec builders. The context did not hold the repo-wide caller set. The second grep in that call failed on the `:!_lym` pathspec.
- **Second `git grep`:** for "status commit" and "records commit" docs and `Rel(` mentions in `shedrun/doc.go`, to find stale prose twins the context did not show.
