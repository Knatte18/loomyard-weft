```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. Trace each group of hunks in the diff to the Goal points of the board entry and to its `Added after Discussion` entry.

**Decisions (Goal points of the board entry, plus the added entry)**
- G1 no root-marker layout read in lymcwd or fabric
- G2 no one-repository clone path
- G3 no `lym-bolt`
- G4 no `--single-repo` flag
- G5 no `hubforge.NewOneRepoHub`
- G6 no path-limited or mid-merge records commits
- G7 no records strip, `Fabric-Merge` trailer or no-fast-forward rule
- G8 no records-only replay
- G9 no prime push guard, and no `operator_unpushed` or `protected` classes
- G10 no statuscommit operation wait, and none of its battencli and loomcli seams
- G11 no `lym-view` ref teardown
- G12 no `lym.primeBranch` key
- G13 the merge-state probe reports a merge only
- G14 a deferred failed-push record is not mailed
- G15 the commit refusal order is the pre-f06acf39 one
- G16 `docs/overview.md` and `fabricengine/doc.go` hold no records-on-code-branch text
- G17 the PATTERN-run-fsm and PATTERN-cwd-resolution entries and backgrounds lose their one-repository text
- Exports: remove the listed symbols; keep `ProtectedBranchRefusal` and the `remote:` lines
- Takes: c83fec49's cached layout probe and its tests go; `FailedPushWayForward` and its callers are weighed before anything is deleted
- A1 (added entry): the peer has landed as c83fec49, so no ordering is needed

**Per change → decision**
- lymcwd `OneRepository`, the root-marker read and `recordsSiblingOwner`, with their tests → G1.
- fabricengine one-repository files (onerepository.go, clone_onerepository.go, recordsduringmerge.go and the rest) and their tests. Residual against f06acf39^ is clean → G1, G2, G3, G6, G7, G8, G9, G11, G12, and Takes (c83fec49 cache).
- fabricengine mergestateactive.go and its cherry-pick test row without the `MergeStateOperation` assertion → G13.
- failedpush.go reverted, and `failedpush_test` gives the deferred record commits → G14.
- commit.go reverted → G15.
- pushclassified.go `ProtectedBranchRefusal`, `rejectionRetryKind`, and the pushclasses protected row → Exports (keep).
- gitrepo: `MergeStartNoFastForward` removed → G7 and Exports. `remote:` lines kept → Exports.
- fabriccli: `--single-repo` flag, `CloneAndWire`'s one-repository push, its doc, help text and tests → G2, G4.
- boardengine: one-repository hub test deleted → G3 and G5 (forced).
- hubforge: `NewOneRepoHub` and the seed → G5.
- statuscommit, battencli and loomcli commit-status seams, with their tests → G10.
- landingshed: `MidMergeRecordsUnsettled` way forwards in finalize.go and premerge.go, with tests → Exports (`MidMergeRecordsUnsettled`).
- landingshed: `Fabric-Merge` expectation in `finalize_integration_test` → G7 (forced).
- loomcli start.go: unsettled kind, with its tests → Exports (forced).
- runfsm `PushOwedWayForward` and the pairteardown caller → G9, the `operator_unpushed` way forward, weighed per Takes.
- `ChangedCodePaths` removed: fabricengine parenttips.go and its test, landingshed deps.go, finalize.go and doc.go, loomcli landingdeps.go, envkit, and both `intentionallyNil` seam tables → forced. It called `RecordsPathsAt`, which G6 and G7 remove.
- cmd/lym allowlist entries and the `MergeStart` pin → forced by the file deletions and the gitrepo change.
- battencli overview_test.go assertion → forced (the layout-read warning is gone).
- PATTERN.md and pattern/: run-fsm and cwd-resolution → G17. single-repo-hub removal, plus the fabric-boundary, lym-files and pair-remote background edits → forced (their text describes removed code).
- README.md and docs/getting-started.md one-repository sections → forced (stale docs). The docs/overview.md statuscommit line → G16.
- fabricengine/doc.go → G16, plus Takes for the cache paragraph.
- The c83fec49 two-repository paragraph removed from fabricengine/doc.go → none (finding 2).
- loomcli wiring_commitstatus_integration_test.go: one-repository rows → G5 and G10. The two-repository subtest "records cherry-pick in progress commits the transition" and its helper `startStoppedCherryPickInRecords` → none (finding 1).
- 91b285da comment rewrap in mergestateactive.go → part of the G13 doc edit.
- A1 → built: the diff starts from a main that holds c83fec49 and deletes its probe cache.

**Per decision → changes**
G1–G17, Exports, Takes and A1 are each built by the changes mapped above. No decision lacks a change.

**Findings**

1. MEDIUM · scope · `internal/loomcli/wiring_commitstatus_integration_test.go`, the removed subtest "records cherry-pick in progress commits the transition" and the helper `startStoppedCherryPickInRecords`.
   - Summary: the diff deletes c83fec49's two-repository seam test, and no decision licenses that. D5 and Takes remove only one-repository tests and c83fec49's layout-probe-cache tests. This test pins G13's two-repository behaviour through the real seam: a records cherry-pick commits the transition.
   - Not forced: `loomCommitStatusDeps`, `newCommitStatusSeam` and `MergeStateActive` all remain, so the subtest would still compile and pass.
   - Evidence: `.lym/burler/review-1883282217/diff/023-internal-loomcli.diff` lines 183–245. Its origin is the `git diff f06acf39 7346d6a8` output for that file, showing c83fec49 added it.
   - Fix: restore the subtest and its helper (and `runGitAllowingFailure` if needed), minus any one-repository use.

2. LOW · scope · `internal/fabricengine/doc.go`, the c83fec49 paragraph after the `MergeStateActive` line (base lines ~1351–1353).
   - Summary: the diff deletes still-true two-repository text: "A weft cherry-pick, revert or rebase does not count; fabric's own replay cherry-picks under the write lock the commit waits on." G16 removes only records-on-code-branch text, and this text states the rule G13 keeps. The same rationale also leaves the `MergeStateActive` doc comment.
   - Evidence: intervening diff of internal/fabricengine/doc.go (c83fec49 hunk at @@ -1348) against the residual, which shows no such lines at HEAD.
   - Fix: keep the two sentences, dropping only the "In a two-repository hub" qualifier.

Tool calls: none. The manifest's residual diffs, intervening diffs and diff pieces covered every trace.
