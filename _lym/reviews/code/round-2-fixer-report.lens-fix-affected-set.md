```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set lens on F8, checking that the change covers every wiring site, claim site and test of the set.

Unchecked findings: none.

no findings

What the searches covered:
- **Deps wiring:** `RangeDiff` is present wherever `ChangedFiles` is: `internal/loomcli/driftwiring.go`, `internal/shedrecipe/entries_drift.go` (seam list), `entries_drift_test.go`, `internal/testkit/envkit/envkit.go`, `internal/battencli/wire_seams_test.go` and the `verify.go` seam list. The other `driftengine.Deps` literals (`internal/driftcli/wiring.go`, `driftRecordsDeps`, `internal/shedrecipe/fixture_test.go`) set no `ChangedFiles` either, so they need no `RangeDiff`.
- **Coverage claims:** `doc.go`, `records.go`, the `rangeRulingKey` and `checkAndJudge` comments, `docs/overview.md:1124`, `contracts/stencils/drift/drift-template-judge.md:58` and the `uncausedReason` comment all agree with a key over the full range diff.

Tool calls:
1. `git grep` for `Deps` constructions and `CommitMessages` sites, because my context had no repo-wide list of the literals.
2. `git grep` for the coverage wording, which failed on a pathspec and returned nothing.
3. `sed` and `grep` over `driftDepsFrom`, the `driftcli` and `fixture_test` literals and the `ReviewBase`/`ChangedFiles` sites, because I had not seen those files.
4. `grep` for the coverage wording again, in place of the failed call 2.
