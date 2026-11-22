```yaml
lens: bound-breaker
outcome: reported
```

I attacked the claims and bounds in the decision record, as the bound-breaker lens.

**Findings**

1. **BLOCKING — D4 / D5: the move note in the body may never reach the fresh run.**
   An entry with a stored Sketch (`discussed: true` plus a record file) runs from its record, never its body.
   `bodyEditRecordWarning` in `internal/boardengine/decision.go` says so: "its run reads that record, never the body, so this body edit does not reach the run".
   The way forward it names is to fold the edit into the record with `lym board upsert --record-file`.
   Counterexample: a discussed entry (motif output, or one filed from a Sketch like `orch-autonomy-mode`) is moved. The orch adds `## Move note` to the body, and the fresh run ignores it.
   D4 has to say that for a discussed entry the note and the archive tag go into the stored record (`--record-file`), or the record is cleared. The record also has no constraint stating this.

2. **MEDIUM — D2's bound: the claim that the snapshot tag keeps the note is unpinned and probably wrong.**
   `internal/fabricengine/snapshotuncommitted.go` says the snapshot holds the weft worktree's uncommitted paths "outside the record pathspec", in the weft repository.
   `_lym/move-note.md` very likely sits inside the record pathspec, so the snapshot would leave it out.
   The cancel agent also reported that `--remove` deletes the run's records.
   So if the orch session dies between reading the note and the upsert, the note can be lost outright.
   The record states the recovery path as fact without checking the pathspec. Either pin it or drop the bound and say the note is lost in that case.

3. **MEDIUM — D2 / D4: the run keeps going while the note is written and read.**
   The creel ask does not stop the run, so the writer and driver carry on between the ask and the cancel.
   Counterexample: Merriam replies, then forks another batch that commits. The tip SHA in the note is now stale, and the work done afterwards is in no note.
   Or the run reaches Finalize and lands before the cancel. D7 then refuses the move, and the operator's intent is silently overtaken.
   D4 needs a hold before the ask, with the bound stated. `lym batten pause` interrupts the session, so the note ask must come after a hold that still lets the writer answer, or the orch must accept the race explicitly.

4. **MEDIUM — D4 step 2: the owner machine can change with the parent.**
   The study says `Owners.Effective(slug, ParentOf)` follows the parent.
   Moving to a beam owned by another machine makes the entry's effective owner that machine. The fresh run then starts there, where the archive tags are fetchable, but the local snapshot from finding 2 is not.
   The record is silent on a move across owner machines. It should refuse that move, or state the bound.

5. **LOW — Constraints: "only `project` records a parent after `--remove`" holds for the parent, but the entry's `owner_machine` field also stays as it was.**
   The run lock never compares `OwnerMachine` (`runlock.go`), so the field is editable. It may need resetting on a move across machines; this ties to finding 4.

6. **LOW — D6: the merge base is taken when the note is written.**
   A Parent-Align merge-in after the note is written but before the cancel makes "merge base to tag" include the old parent's merged content.
   Finding 3's hold also closes this.

**Claims checked**

- The cancel sets the entry back to `draft` and frees the run lock: held (`runlock.go`, `clearRunClaim`).
- Only `--remove` makes archive tags, and cancel reports them as `archive_tags`: held (`cancelcmd.go`).
- Cancel refuses on a machine that does not own the entry: held.
- The cancel's refusals guard landed work (D7): held. The race in finding 3 is a separate issue.
- The release stays with the new parent's orch: held (orch-role text).
- The snapshot tag keeps the note: broken or unpinned (finding 2).
- The body is the durable copy that the fresh run reads: broken for discussed entries (finding 1).
- Ordering note → cancel → body is safe against an advancing run: broken (finding 3).
- The open risk on `stale-records-ref-blocks-rerun`: held.

**Tool calls**

1. Grepped pairteardown and fabricengine for snapshot handling and boardengine for `record_body_sha256`: the context did not hold the snapshot scope or whether the body reaches a run.
2. Read the staleness rule in `internal/boardengine/decision.go` and grepped `.gitignore`: needed to confirm a discussed run ignores the body.
3. Grepped `internal/fabricengine/snapshotuncommitted.go` for whether untracked and record-pathspec paths are in the snapshot: the context only had the subagent's summary.
