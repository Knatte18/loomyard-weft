```yaml
lens: fix-affected-set
outcome: reported
```

Findings below come from my three searches, one per finding with a set premise (F1, F2, F3). I left F4 and F5 unchecked because neither has a set premise.

**F1 — repo-side literals.** The fix covered the two literals the review named but not the rest of the set.
- LOW: `internal/loomcli/prepush_test.go:35` still builds `boltMain` with the literal `Repo: "bolt"`. The fix exported `RepoCode` and `RepoRecords` but no bolt constant, so `failedPushSideBolt` stays unexported and the test repeats `"bolt"`. A rename on the fabric side would leave this row silently wrong.
- LOW: `internal/loomcli/commitrecords_test.go:119`, `:123` and `:127` build `FailedPush` with the literals `"records"` and `"code"`. The review's `location` named only `prepush_test.go`, but the review's own premise (loomcli tests repeating the fabric literal) covers these too. They were left as they were.
- Non-production sites in other packages (`hibernatecli`, `battencli`, `adoptcli`, `orchcli`, `landingshed`, `fabriccli`) also use the bare literals. They are outside the finding's loomcli scope, so I raise nothing on them.
- No production literal for the repo side remains in loomcli. `parentOutcome` and `stoppedBranches` use `fabricengine.RepoCode` and `RepoRecords`.

**F2 — run paths outside `internal/shedrun`.** Covered, no finding.
- The search of non-test code for `filepath.Join(...ScratchDir(` outside `shedrun` finds nothing in `loomcli`. The only `loomcli` hit, `driverreport.go:32`, joins a name onto `shedrun.DriveReportsDir`, which predates the fix.
- `parentNoticeFile` is gone and `ParentNoticeFile` has a row in `TestPathConstructors`.

**F3 — wiring of the check to its seams.** Covered for the sites the review named, plus one out-of-scope note.
- Every push or commit site in `loomcli` behind the status deps and `pullRequestDeps.PushTask` now goes through the injected check: `wiring.go` Commit, Spawn and Push, and `landingdeps.go:343`.
- LOW, out of the finding's scope: other `loomcli` run-owned commits and pushes never pass the check. These are `landingdeps.go:125` (status checkpoint commit), `landingdeps.go:651` (landing checkpoint commit), `drift`, `circling`, `projectrecord`, `decision` and `sharedbootstrap`. They are commits and not pushes, which fits the guard's stated scope (`guardedCommit` is for "a run commits for a push"), so I raise it only so the fixer-report can say it was seen.
- `battencli` (`commitstatus.go:53`, `cleanup.go:84`) and `tendercli/deps.go:148` spawn status pushes unguarded. The review's PATTERN bullet says "A loom run's commit and push", so batten's runs are out of scope.

**Calls made (3 of 6):**
- Grep of `internal` for `Repo` literals `"code"`, `"records"` and `"bolt"` outside `fabricengine`: my context showed only the tests I had read, not the full set under F1's key.
- Grep of non-test code for `filepath.Join(` over `ScratchDir` or a `shedrun` directory function outside `shedrun`: my context could not show whether other run paths were built outside `shedrun`.
- Grep of non-test code outside `fabricengine` for `PushPairAnchored`, `SpawnStatusPush`, `CommitAnchoredPaths` and `PushTask:`: I needed every push and commit site to check F3's coverage.
