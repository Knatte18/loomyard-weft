```yaml
lens: bound-breaker
outcome: reported
```

I attacked the record's bounds, guarantees and claims about existing code. Two claims break and three need tightening.

**Findings**

1. **MEDIUM — D2: the tracked copy can end up older than the live record.**
   - The claim: "the copy is overwritten whole each time, so the last copy is the record's final state."
   - The live ask record changes through `state.UpdateJSON` under its lock, and more than one process writes it: the run's loop asks, expires and settles, and the parent's `lym loom parent-review answer` answers.
   - The record does not say the copy is written under that lock. If it is written after the lock is released:
     1. Process A updates the record and releases the lock.
     2. Process B updates it again, copies it, and releases.
     3. A then copies the state it read before B's update.
   - The tracked copy now holds a stale state, and D2's acceptance criterion ("matches the live record's final state") fails.
   - Fix: D2 states that the copy and its report copies are written inside the same lock as the record change (in the `UpdateJSON` closure or under `files.Lock`), with an atomic whole-file write.

2. **MEDIUM — D5: `--extract` picks "the slug's newest run", but one run can have two tags.**
   - `fabricengine` teardown tags the local weft tip. When origin's tip has diverged from it, it also tags origin's tip.
   - So one run can leave two `archive/<slug>/*` tags.
   - "Newest" by commit time can pick the origin-tip tag. That tag lacks the kept files and the index row that were committed locally at teardown.
   - Fix: define the order. A tag holding an index row comes before one without, and tip commit time breaks ties. Each line should also show the tag's tip time, so `--tag` can choose between them.

3. **LOW — D5's bound leaves out that the list is only as fresh as the last fetch.**
   - `reportengine.Open` lists tags with `records.TagRefs(fabricengine.ArchiveTagPrefix)`, which reads local refs only.
   - Because the verb "never fetches", runs archived on another machine appear only after the records repository fetches tags.
   - The bound should say so, so a study on one machine does not take missing runs as lost.

4. **LOW — D3: a fixer that does not end leaves no copy.**
   - `landingshed` writes the report at `<ScratchDir>/<row>-fixer-report.md`, a fixed name per row that each attempt overwrites (`fixerrun.go`, `fixreview.go`).
   - D3 copies it only "once the fixer ends". A fixer that is killed, or whose run crashes before the copy, loses that attempt's report.
   - The attempt-number naming also has to come from the row's attempt count, not from the file name.
   - Fix: say where the attempt number comes from, and copy on every fixer end, including a failed or timed-out one.

5. **LOW — D6 cites the wrong kind of symbol.**
   - `fabricengine.PlainArchiveTagsOnOrigin` lists legacy plain tags; it does not delete anything.
   - The exception should name the cleanup step that deletes them, or describe it without naming a symbol.

**Claims attacked**
- D1, "every findings file passes through `pr-findings`": held for the agent's PR-Review findings. The parent's PR-Review findings, the `/tmp` case, go through `parent-review answer`, which parses them into `ask-<n>.report.yaml` (`writeParentReport`), so D2 covers them.
- D1/D2/D3 criterion that a kept file written late is in the tag: held. `fabricengine` `remove.go` runs `commitPendingRecords` before `tagWeftTipForArchive`, and `force` never skips it. The cleanup sweep in `cleanup.go` tags without that commit, but it only acts once the records worktree is gone, so nothing is pending then.
- D2 claim about the older `parent-review/round-*` reader: held, given a distinct directory.
- D4 "resumed Finalize rewrites the same row" and "a cancelled run writes none": held, since the row is keyed to the landing commit.
- D5 "derived row for older tags": held. `Open` already lists tags and joins landings. The `archive-owner/` hand-over tags are not matched by the `archive/` prefix, which is correct.
- D6 "no code deletes an archive tag": held. No tag-delete primitive exists in `gitrepo` or `fabricengine`, and `prune.go` says it has no archive step.
- D5 bound "writes only under `<dir>`, never writes a ref": held as designed.
- Constraint "`_lym` is the whole committed set": held (`structuralCommittedDirs`).

**Tool calls**
1. Grepped `fabricengine` for the archive-tag and pending-records commit sites, and the code for any tag deletion. My context lacked whether teardown commits before tagging on every path, and whether any tag delete exists.
2. Read `cleanup.go` around its call to `tagWeftTipForArchive`, and `remove.go`'s pending-commit step. My context lacked whether the cleanup sweep commits pending records.
3. Grepped `reportengine/source.go` for how tags are listed. My context lacked whether `Open` fetches or reads local refs only.
4. Grepped `fabricengine/prune.go` and the git client for tag or ref deletion. My context lacked whether prune touches archive tags.
5. Grepped `landingshed` for the fixer report path. My context lacked whether the report name is per attempt or fixed per row.
