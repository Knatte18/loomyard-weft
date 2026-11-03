---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: internal/loomcli/wiring.go:957
    summary: The commitStatusDeps call site in wire(), the one a real motif run steps through, is pinned by no test; only the wireLightweight site is driven.
    fix: In internal/loomcli/commitstatus_integration_test.go, run TestWireLightweight_CommitStatusWritesFinishedStatus's table once per wiring entry point, wireLightweight and wire (c.wire(location, location.AnchorPath()), as wiring_publish_integration_test.go does against a hub pair), then call c.shedPaths.CommitStatus("Discussion-Bouncer", "done") and assert the same board status per recipe. Rename the test to drop "WireLightweight", and give each row a distinct slug per entry point so the shared hub's pairs do not collide. Replacing the wire() call with loomCommitStatusDeps(location, c.runID) must then fail the motif row.
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/battenshed/land.go:157
    summary: Land, Worktree-Teardown and the landing notice treat a Finalize-less child (a motif run) as a landed one, and no test in battenshed or battencli drives that end state.
    fix: Give LandDeps a seam, for example ChildLanded func() (bool, error), that battencli fills from the task worktree's status history (a Finalize entry with outcome Done, the check pairteardown's childLanded makes). In landProducer.Call, when it reports false, skip waitCleanPrime, pullAndDeploy and recordsStopped (so no deploy command runs and no "landed" notice is posted for a run that landed nothing) and still call PruneBoard, then return Done. Add rows to internal/battenshed/land_test.go for a child with no Finalize (no pull, no deploy, no notice, Done) and one with a Finalize (unchanged behavior). Add one battencli wiring test that the seam reads false for a status history with no Finalize row. If the parent decides Land should stay as is for such a child, record that with lym loom decision add and add the land_test.go row pinning that Land pulls and deploys for it.
  - id: F3
    severity: LOW
    class: scope
    location: internal/loomcli/discussionrecord_integration_test.go:123
    summary: TestSeedDiscussedRecord_Real pins only the discussion-directory failure; the board read failure, the record write failure and the commit failure of seedDiscussedRecord return paths have no row.
    fix: Add rows to the TestSeedDiscussedRecord_Real table. One makes record.path a directory, so the record write fails and the error reads "write the stored decision record". One makes the board read fail, for example by making the flagged entry's decision-<slug>.md in fabricengine.BoardDir a directory, with wantErr and the expected message fragment. One makes the commit fail, for example by leaving the pair mid-merge. Generalize wantErr from the record.dir-only check to a per-row expected message fragment (an errContains field), and keep every error row asserting that the returned seeded value is false and that no support log was written for the board failure.
---

### [MEDIUM] The wire() commit-status call site is untested

**Location:** internal/loomcli/wiring.go:957
**Issue:** `commitStatusDeps` is called at two sites: `wireLightweight` (wiring.go:319) and `wire` (wiring.go:957).
`TestWireLightweight_CommitStatusWritesFinishedStatus` drives only the first, and `TestRecipeKeyedWiring` calls the method directly, so neither sees which constructor `wire` uses.
Replacing the `wire` call with `loomCommitStatusDeps(location, c.runID)` leaves `FinishedStatus` empty for a motif run and fails no test.
`wire` is the entry the run-stepping verbs use, so it is the site that decides whether a finished motif run leaves its entry as draft or leaves it under the run lock, which is the behavior the commit message of 1160a98c adds.
`wiring_test.go` checks only that `c.shedPaths.CommitStatus` is non-nil after `wire`.
The `SeedDiscussedRecord` line, by contrast, is pinned: it is not in `intentionallyNil`, so deleting it makes `TestWire_EverySeamFilled` fail.

### [MEDIUM] A Finalize-less child is handled as a landed one in batten, with no test

**Location:** internal/battenshed/land.go:157
**Issue:** A motif child ends with `Discussion-Bouncer` done and no Finalize row, so it makes no landing commit.
Batten's graph still runs Run-Shed, then Land, then Worktree-Teardown for it, and none of the three reads whether the child landed.
Land (land.go:157-219) pulls the prime and, when the parent is a deploy branch, runs the hub-wide deploy command, then posts the records-stop notice worded "landed".
For a run that the decision record says "lands no code", the deploy runs with nothing new to deploy, and a records stop reads as a landing.
Worktree-Teardown is sound for this end state: `RemoveRefusal` refuses only on dirtiness and merges in flight, and `settleClaim` keeps the draft entry (pinned by the new `TestSettleClaim_DraftEntryOfFinalizeLessRun`).
But `teardownRequest` sets `Landed: true`, so the only thing between a motif entry and `done` is that its status is draft; had the board status write at `newCommitStatusSeam` failed (it only warns), the entry would be marked done.
The diff changes battenshed and battencli only in comments and tests of names, so no test in either package drives a Finalize-less child through Land, the notice or the teardown request.
The rubric item is scope against the answer key: the amended entry asks that a motif run lands no code, and the batten path that follows the run still treats it as a landing.

### [LOW] seedDiscussedRecord's failure returns are mostly unpinned

**Location:** internal/loomcli/discussionrecord_integration_test.go:123
**Issue:** `seedDiscussedRecord` has these returned errors: board open, `DecisionRecord` read, discussion directory creation, record write, support-log stat, support-log write and commit.
The table covers the directory creation (`UnwritableDiscussionDirectoryFails`), the existing-record, existing-log and missing-record-file cases, and the happy path with its commit.
The record write, the board read and the commit failure each return a different error the gate turns into a failed run, and none has a row.
Support-log stat and write failures are of the same kind and cheap to leave out; the three named in the fix are the ones with distinct messages.

## Rejected

- Glossary (`docs/research/weaving-glossary.md`): the recipe-name list, the `shed` row's recipe list and the `motif` row all name motif as a recipe a board task may name, running Discussion alone and run inline by tapestry as its Motif block; the `arras` row reads as free again and `cartoon` stays reserved for a later Plan block. Not a defect.
- The `SeedDiscussedRecord` line in `wire`: deleting it fails `TestWire_EverySeamFilled` (the field is absent from `intentionallyNil` and `envkit.NilSeams` reports nil seams). Not a defect.
- The `wireLightweight` commit-status call site: replacing it with `loomCommitStatusDeps` fails `TestWireLightweight_CommitStatusWritesFinishedStatus` on the motif row. Not a defect.
- `TestSeedDiscussedRecord_Real` rows for a record or support log already on disk and for a flagged entry whose record file is missing: both are present and assert the right outcome (record overwritten, log kept, nothing written).

## Scope of this review

I read the focus files, the decision record, `wiring.go` up to line 974, `discussionrecord.go`, the glossary diff, the battenshed Land, teardown and notice sources, the pairteardown claim source and the batten recipe, and the diff of the landing commits' battenshed, battencli, pairteardown, loomshed and shedbuild/loomcontract files.
The rest of the 94 changed files, among them the recipe YAML, the expansion and the snapshot test in `loomrecipe`, were not read this round beyond the stat and the commit messages, and no test or build was run.
