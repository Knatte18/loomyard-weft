```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the record's states (the move note file, the entry body, `project`, the archive tags, the re-run of the same slug) against the code that already writes or reads them.

**Findings**

1. **BLOCKING — D4 / D5 (body as the carrier of the move note).** For a discussed entry, the fresh run never reads the `## Move note` the orch writes into the body.
   - Evidence: `boardengine.Board.bodyEditRecordWarning` (`internal/boardengine/decision.go`) says an entry with `discussed: true` and a stored record file "has a stored decision record, and its run reads that record, never the body, so this body edit does not reach the run".
   - The way forward it names is to fold the edit into the record and store it with `lym board upsert … --record-file`.
   - This covers motif and arras entries and every entry with a Sketch. This task is itself `recipe: motif`.
   - The record is silent on this. The fix: D4 step 3 folds the move note into the stored record with `--record-file` when the entry is discussed, and writes it to the body only when it is not. D6's "read the diff" instruction must reach the run the same way.

2. **MEDIUM — D2 bound ("never committed by the writer; `--remove` keeps the uncommitted note under the local snapshot tag").** The bound holds, but I found a path the record does not consider.
   - `_lym` in the task worktree is a link to `<slug>-weft/_lym`, so the note lands in the records worktree. Cancel's own commit (`commitCancelRecord` → `deps.Commit`) commits batten's record on the prime's records side, not the task weft, so it does not pick the note up.
   - `gitrepo.StageAllAndCommit` (`git add -A`) and `fabricengine` `ownerarchive.go`'s `add --all` exist, and any park or teardown commit through them would also commit the note onto the records tip that `--remove` archives.
   - That outcome is harmless, and better than the record claims. The real gap is the other direction: the claim "recoverable there and nowhere else" may be wrong.
   - Suggested fix: drop the "nowhere else" wording, or state that the records archive tag may also hold the note.

3. **MEDIUM — D4 step 2 / D5 (re-run of the same slug).** The re-run path meets more leftovers than the stale tracking ref in Open risks.
   - `--remove` leaves `archive/<slug>/…` tags and a `snapshot` tag under the same slug.
   - `fabricengine` `originreconcileended.go` passes `EarlierSnapshot: true` to teardown when the weft repository "already holds a snapshot tag of the slug", and `pairteardown/originseams.go` branches on it.
   - `battencli/readopt.go`'s `Tags` seam (`fabricengine.SlugRecordsTags`) lists every archive and set-aside tag of the slug during a resume of a pair whose task worktree is gone.
   - A fresh run of the same slug therefore meets the old run's tags on these two paths. I could not trace whether either misreads them; the record states no claim either way.
   - Suggested fix: name these two paths in Open risks beside `stale-records-ref-blocks-rerun`, or settle them now.

4. **LOW — D4 step 3 (`project` change on a draft).** Survives. `checkRunLocks` compares scope only on run-status entries, and cancel leaves `draft`.
   - `parent_beam` is derived by `ParentBeam`, and `origin.json` is gone after `--remove`, so `ParentOf` falls back to the board's `project`. Nothing re-writes the old parent.

5. **LOW — D4 step 4 (release).** Survives. `set-status` with null releases it, and the dobby of the new parent's worktree starts it, since the derived `parent_beam` names it.

**Paths traced**

- D1: cancel's run-lock clearing (`clearRunClaim` → `draft`), survives.
- D2:
  - `_lym` link to weft, note lands in the records worktree.
  - `commitCancelRecord` (prime side), does not commit the note.
  - The snapshot in `pairteardown` `teardown.go` `takeSnapshot`, keeps it.
  - `StageAllAndCommit` / `ownerarchive` `add --all`, may commit it (finding 2).
- D3: no state of its own, nothing traced.
- D4/D5:
  - `bodyEditRecordWarning` / stored record read, breaks for discussed entries (finding 1).
  - `checkRunLocks`, survives.
  - `ParentBeam` / `ParentOf` fallback, survives.
  - `EarlierSnapshot` teardown branch and `readopt` `Tags`, unknown (finding 3).
- D6: the archive tags are pushed by `--remove` and read via git only, survives. Delivery to the run is as in finding 1.
- D7: cancel refusals unchanged, survives.
- D8: the glossary is a text-only addition, nothing traced.

**Tool calls**

1. `ls _lym` and grep for Snapshot in `pairteardown`/`fabricengine`: to learn where `_lym/move-note.md` lands and how `--remove` keeps uncommitted files.
2. `originreconcile.go` and the `EarlierSnapshot` grep, plus cancel.go record grep: to learn which leftovers a same-slug re-run meets and what cancel removes.
3. grep `commitCancelRecord` in `cancelcmd.go`: to learn which paths cancel commits.
4. `sed` of `commitCancelRecord`: same reason, the body of that function.
5. grep for `record_body_sha256` users: to learn what a body edit does to a discussed entry.
6. `sed` of `decision.go` and `board.go` spans: same reason, the warning's text and conditions.
7. grep for `add -A` / `add --all`: to learn whether any commit path stages the whole records tree.
