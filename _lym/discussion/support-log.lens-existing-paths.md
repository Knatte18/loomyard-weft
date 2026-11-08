```yaml
lens: existing-paths
outcome: reported
```

Existing-paths lens: I traced the code paths that already touch the state each decision introduces or changes. Two findings are MEDIUM and three are LOW; I found nothing BLOCKING.

**Findings**

1. **MEDIUM, D2: the copied ring wiring would write into the Sketch record.** D2 says the ring is "wired as `motif-recipe.yaml` wires the Discussion ring" but never says which of that ring's keys it drops.
   - That ring sets `carry_over: Discussion-Review`, which files a round's still-open findings into the decision record's Open risks. A split perch copying it would change the Sketch record after the proposal was written, and Raddle-File's `risk-uncarried` check reads that record.
   - That ring also sets `approve_seam: discussion`. Copied onto `Raddle-Bouncer`, it would make the perch a second Discussion approval row. `loomrecipe.PastDiscussionApproval` and `DiscussionApprovalRow` look up that row on the expanded recipe, so `lym loom decision add` would no longer tell where the run is relative to the Discussion approval.
   - Fix: D2 states that the split perch sets neither `carry_over` nor `approve_seam`.

2. **MEDIUM, D2: committing the fixed proposal needs a new seam, and the record does not say how it squares with the PATTERN rule.** D2 says "the ring commits a fixed proposal the way Raddle-Write commits the raddle directory".
   - The Bouncer's `commit_seam` only accepts `plan` or `discussion`, each mapped to an `Env` closure (`env.CommitPlan`, `env.CommitDiscussion`) in `internal/shedrecipe/entries_bouncer.go`.
   - A raddle commit would therefore be a third `Env` closure. That runs against `PATTERN-shed-recipe-registry` ("A row never gets its own Env field").
   - Fix: say whether a commit seam counts as a row value under that rule, or list a PATTERN change.

3. **LOW, D8: the other ways to release don't release the members.** D8 changes only `lym board set-status` with a null status.
   - `boardengine/doc.go` lists two more release paths: an upsert setting `status` to null, and a merge whose `set_status` sets null.
   - Under D8 those would release the project but not its members, so the same release gives different results depending on the verb.
   - A null status on a done project would also reopen it, and D8's bound doesn't cover that case.

4. **LOW, D6: decision verbs after the filing.** `lym loom decision store` refuses an entry that is done (`ErrRecordEntryDone`). A converted project is never done, so a decision added after the filing overwrites the project's whole stored record. That is consistent with D6's intent, but the record doesn't mention it.

5. **LOW, D1/D2: Raddle-File's return re-enters the perch.**
   - A return after approval re-enters `Raddle-Bouncer` over a changed proposal. `shedadapters/bouncer.go` already starts a new cycle in that case, so the perch survives.
   - The record says nothing about whether the segment's bounce budget carries over across returns.

**Paths traced, by decision**

- **D1**
  - The `undiscussed` gate on restart: survives. The record is seeded from the board.
  - `finishesAsSketch` from both status seams: survives. The recipe holds a `RaddleFile` row, so no Sketch is made.
  - `PastDiscussionApproval` on the expanded block: survives, provided finding 1 is fixed.
- **D2**
  - Bouncer re-entry over a changed subject: survives.
  - `commit_seam` closed set: breaks (finding 2).
  - `carry_over` / `approve_seam` copied from the Discussion ring: breaks (finding 1).
- **D4**
  - Raddle mode recorded on the first call and kept in state across returns: survives.
  - Cold proposal stored by the gate's onPass in auto mode: survives. Later fixer edits don't touch the stored cold copy.
- **D5**
  - `EndRaddleEntry`, then the landing's `PruneBoard`: survives. The done entry is removed.
- **D6**
  - Run-status writes from `loomcli.commitStatusDeps` and from batten's commit-status seam (`battenCommitStatusDeps`), both calling `boardengine.Board.SetRunStatus`: covered by D6's skip.
  - Dobby's `released` and `readyToStart`: survive. They read kind task only, so the project is never started.
  - `Store.Prune`: survives. A project that isn't done is kept.
  - `raddleFilingHeld` on retry: survives. The body opens with the project prefix.
  - `lym loom decision store` / `add` after the filing: survives (finding 4).
  - The landing's `childFinalized` false: no board write found on that path, so it survives.
- **D7**
  - `decision add` past the approval with no Record-Write row: survives. `rowWritten` is false, and `copyRecordToBoard` keys on `pastApproval`.
- **D8**
  - `set-status`: changed as the record says.
  - The upsert null-status and merge `set_status` releases: they don't release the members (finding 3).

**Tool calls**

1. Read `internal/battencli/commitstatus.go` and `internal/loomcli/wiring.go` around the status seams, because my context lacked the status writers that run after Raddle-File.
2. Read `internal/loomrecipe/discussionapproval.go`, because my context didn't show whether the approval-row lookup expands blocks.
3. Read the spans of `internal/loomcli/decision.go` that run past the approval, because my context lacked what `decision add` and `decision store` do there.
4. Read `internal/dobbyengine/starts.go` around `released`, because my context didn't show whether dobby starts project entries.
5. Read `internal/shedrecipe/entries_bouncer.go` and grepped `internal/shedadapters/bouncer.go` for re-entry, because my context lacked the closed seam values and how re-entry is handled.
6. Grepped `internal/battencli` and `internal/battenshed` for what landing does to a run with no Finalize, because my context lacked any board write on that path.
