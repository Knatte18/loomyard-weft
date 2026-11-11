```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the code paths that already write or read the state each decision touches.

**Findings**

1. **BLOCKING, D5: the promote warning fires on every Sketch a motif run makes.**
   - Discussion approval stores the record, which under D5 stamps `record_stored`.
   - Later, `sketchFinish` → `MakeSketch` (`internal/boardengine/sketch.go`) changes the entry's `Kind` from task to note and its `Labels` (it adds `sketch`).
   - `scopeChanged` (`internal/boardengine/runlock.go`) counts both `Kind` and `Labels`, so `stampDates` (`internal/boardengine/dates.go`) moves `revised` past `record_stored`.
   - The result: a fresh Sketch with an untouched body warns at its first promote.
   - The bound in D5 names title and label edits by a person, not this path every Sketch takes.
   - Fix options: `MakeSketch` stamps `record_stored` too, or the comparison leaves out the sketch conversion.
2. **MEDIUM, D5: the claim that every write of a stored record stamps `record_stored` misses writers.**
   - `internal/boardengine/raddlefile.go` writes `decision-<slug>.md` directly, at lines 190 and 214. That covers filed parts and the arras project's whole Sketch record. It does not go through `Board.StoreDecisionRecord`, so those entries never get `record_stored`.
   - `replay.go` replays `decision-<slug>.md` files during a board sync as entry changes, so a record written on another machine arrives through that path.
   - Two consequences:
     - Raddle-filed entries silently fall into the "no `record_stored`, no warning" case, although the bound only names entries stored before this task.
     - Replay must carry `record_stored` together with the file.
3. **MEDIUM, D1 `--amends-plan`: the amendment record's titles have a reader the record does not trace.**
   - `AmendmentRecord.Titles` describes "pending amending decision entries", and `internal/shedadapters/bouncerdecision.go:337` reads them.
   - The record does not say what happens in these two cases:
     - A replace title with no `### Added after Discussion` entry behind it.
     - A replace that folds away an Added entry whose title an earlier `add --amends-plan` already put in the amendment record.
   - In either case the Plan-Bouncer's amendment round may look up an entry by title and not find it.
   - I did not read past line 337, so this is unverified. D1 should state how a replace's amendment title is reviewed, and what becomes of pending titles whose entries it folded.
4. **LOW, D2/D3: a re-seed after a failed board copy.**
   - `seedDiscussedRecord` (`internal/loomcli/discussionrecord.go`) writes the board's stored record into the run and commits it when the run enters the Motif block on a discussed entry.
   - Its commit carries no replace trailer.
   - Failure case: D3's board copy fails (a `record_store_error`, which is not a refusal), and the run later re-enters the block through a goto or restart.
   - The seed then overwrites the replaced record with the stale board copy. Any entry added after the replace would then be an `added-decision-dropped` finding.
   - The record is silent on this. It should say that the way forward from a `record_store_error` is `decision store` before any re-entry.

**Paths traced, by decision**

- **D1, replace writes the record:**
  - `checkDarnRecord`, its working-equals-committed precondition: survives.
  - `copyRecordToBoard`: survives, given D3.
  - `notifyLiveRuns`: survives.
  - `documentSubject`/`IdentityText`: changes the review subject, which D1's bound states. Survives.
  - The amendment record and `bouncerdecision.go`: unclear (finding 3).
- **D2, the bounded history:** `recordHistory` has four callers: `wiring.go:418` (validate env), `wiring.go:881` (gates), `darnwiring.go:165` (Darn-Write guard) and `decision.go:1121` (add/replace).
  - Bounding `recordHistory` reaches all four, and `CommittedAnchoredFileHistory` has no other caller. Survives.
  - `seedDiscussedRecord` commits without a trailer: survives, except in finding 4's case.
- **D3, the board copy for runs whose entry carries a stored record:**
  - `StoreDecisionRecord`'s run-lock bypass: survives.
  - Raddle-File reads the run's record path, not the board: survives.
  - `perchFilingRefusal` still gates both verbs: survives.
- **D4, `--record-file`:**
  - `checkRunLocks` compares only `board.json` fields, so it cannot see the record file. The new refusal must be a separate check, which the plan-writer note implies. Survives.
  - `replay.go` treats `decision-` files as entry changes: survives, but `record_stored` must replay with the file (finding 2).
- **D5, `record_stored` and `revised`:**
  - `MakeSketch` with `stampDates`: breaks (finding 1).
  - `raddlefile.go` record writes: not stamped (finding 2).
  - Promote's own `Kind` change is excluded via `promotedSlug`, and the comparison reads the before-state: survives.

**Tool calls**

- grep of `CommittedAnchoredFileHistory`/`recordHistory` callers and the seed's write and commit: my context had only `projectrecord.go`.
- grep of `scopeChanged`/`stampDates` and `MakeSketch`'s field changes: the board explorer's report named `Revised`'s trigger but not which fields count.
- grep of `DecisionFileName` users and board deferral: to find every other writer of the record file.
- read of `replay.go` and the `stampDates` body: to see how replay and date stamping treat record files.
- grep of `amendment.go` for title and Added ties: `--amends-plan`'s consumers were not in my context.
- grep of `.Titles` consumers: to find who reads the amendment titles.
