```yaml
lens: existing-paths
outcome: reported
```

existing-paths lens: tracing the existing code paths that write, reset or read the state D1 to D6 add.

**Findings**

1. **MEDIUM — D1, D2: the new directories under the reviews root show up as fake review segments in `lym report`.**
   `reportengine` `RunRow.readSegments` (`internal/reportengine/rounds.go`) lists every subdirectory of `LoomReviewsDirRel()` and adds each one, except `parent-review`, to `row.Segments` as a review segment, counted by `shedadapters.ReviewRounds`.
   D1's PR directory and D2's ask-copy directory would therefore appear as segments in every report verb that renders `Segments`, such as convergence.
   The record only keeps D2's directory apart from the older `parent-review` reader. It says nothing about this walker.
   Fix: put the copies outside the reviews root, or have D1 and D2 say that `readSegments` skips them, with a criterion for it.

2. **MEDIUM — D3, D4: files under `_lym/landing/` are not in the status commit, and while uncommitted they block the batten binary re-step.**
   `statusCommitPathspec` (`internal/loomcli/wiring.go`) stages only the status file, the reviews dir, the loom durable dir and the driver records.
   `loomengine.PathsOutsideRunRecords` names the same set.
   - D3's fixer copies and D4's index row are not under any of these, so no transition commits or pushes them. They reach origin only through teardown's commit of uncommitted records.
   - Until then they count as dirty paths outside run records. `battencli` `dirtyPathsOutsideRunRecords` feeds `ChildDirtyPaths`, and `battenshed` `innerrun_binaryrestep.go` then skips re-stepping a halted child on a new build ("the task pair has uncommitted paths").
   - A Finalize that halts after a landing fixer ran, which is D3's normal case, would therefore lose the auto re-step.
   - If the owner machine dies before teardown, the copies never reach origin.

   The record claims next-commit coverage only for D1, and is silent on both effects.
   Fix: add the landing copies to the status commit pathspec and to `PathsOutsideRunRecords`, or place them in a dir already in that set, and add a criterion.

3. **LOW — D2: copies are written from another process while a status commit may be staging.**
   `AnswerParentAskOrUnreached` and `ExpireParentAsk` run in the parent's or the dobby's process, not the driver's.
   D2's "overwritten whole" copy should be an atomic write (temp file and rename, as `internal/state` does), so a concurrent status commit never stages a half-written copy.
   The record doesn't say so.

4. **LOW — D4: "a resumed Finalize leaves the same row" holds only if the landing time comes from a durable source.**
   That source would be the merge record or the landing commit. A resumed Finalize that stamps the current time breaks the D4 criterion.

**Paths traced, per decision**

- **D1:**
  - Status commit stages the reviews dir: survives.
  - `report` `readSegments`: breaks (finding 1).
  - `arbiterstop.NewReaders`, `circling`, `resume` and `hardenwiring` read only named Bouncer or harden subdirs: survives.
- **D2:**
  - Same reviews-root paths as D1: breaks on `readSegments`.
  - The live `.lym/parent-review` record's lock: survives.
  - Concurrent writers outside the driver: LOW (finding 3).
  - `archivedrecords` older-round reader: survives, since the directory is distinct.
- **D3:**
  - Status commit pathspec: not covered.
  - Batten binary re-step dirty check: breaks (finding 2).
  - Teardown commit: survives.
  - `landingshed` `fixerattempts.go` removes only its attempts file, not reports: survives.
- **D4:**
  - Status commit: not covered.
  - Dirty check: breaks while uncommitted after a post-landing halt.
  - Resume: LOW (finding 4).
  - Teardown commit and tag: survives.
- **D5:** read-only over tags. Its readers are `reportengine.Open` and the landings join, and no writer meets it: survives.
- **D6:** the only deleter of `archive/` tags is the older plain-tag cleanup, which the record names: survives.

**Tool calls**

1. Grepped `RemoveAll`/`os.Remove` in the loom and landing packages, and the reviews walker in `rounds.go`: my context lacked who deletes or walks review and landing files.
2. Read `readSegments` and the users of `LoomReviewsDir`: to see how the reviews root is enumerated.
3. Read `PathsOutsideRunRecords`, the arbiterstop and circling readers, and `statusCommitPathspec`: to learn which dirs the status commit stages.
4. Traced the callers of `PathsOutsideRunRecords` in `battencli`: to learn what consumes "outside run records".
5. Read the `ChildDirtyPaths` wiring: to find what the dirty check gates.
6. Read the `battenshed` binary re-step consumer: to confirm the effect of a dirty path.
