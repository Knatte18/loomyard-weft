```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun F1's search for every site that builds a run-records commit pathspec, and check that F2's doc edits cover their set.

F1's set is the commit pathspecs that stage a run's records. I searched all non-test callers of `CommitAnchoredPaths`.
- **Covered:** `statusCommitPathspec` (`internal/loomcli/wiring.go:99`) and `battenRunCommitPaths` (`internal/battencli/commitstatus.go:68`) both stage the settle log now. `commitRecordsVerb` reuses `statusCommitPathspec`, so it is covered too.
- **Stages the whole root:** `internal/loomcli/landingdeps.go:636` commits `runsRoot`, so the settle log is carried there.
- **Status only:** `internal/loomcli/landingdeps.go:125` commits only `shedrun.StatusRel`, a "status checkpoint". It is a landing-time checkpoint, not a status or records commit, so I count it out of the set.
- **Not checked:** `internal/battencli/cleanup.go:80` builds its own `relPaths`. I didn't read it, so I can't say whether it stages the settle log.

## Findings

### LOW: `loomCommitStatusDeps` doc is stale
- **Changed text:** `internal/loomcli/wiring.go:197`, the `loomCommitStatusDeps` doc.
- **Issue:** It lists what `statusCommitPathspec` stages as "the status file, plus the review round record, the loom durable directory and the drive reports". That omits the arbiter and tackler directories and now the settle records, though the pathspec's own doc at line 85 was updated.
- **Evidence:** the line 197 text against the line 85 doc. The settle-records half of the gap comes from this fix, which updated line 85 and left line 197.

## F2's set
Searching for the stale phrases found no other site: "dirty-path check counts", "run-records root" and `RunsRootRel` appear nowhere else with stale text. The `SettleRecords` and `RunsRootRel` doc edits cover their set.

## Tool calls
- `grep` for `CommitAnchoredPaths`, `DriveReportsRel(` and `TacklerRel(` in non-test Go files: the context lacked the full list of commit-pathspec sites.
- `grep` for the stale phrases and `RunsRootRel` across Go and markdown files: the context lacked the F2 set of docs mentioning these.
