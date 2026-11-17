```yaml
lens: existing-paths
outcome: reported
```

Task: existing-paths lens. I traced the code paths that already write, reset or read each state the record introduces or changes.

**Paths traced, per decision**

- D1, fixer-report frontmatter:
  - `skipFix`, which writes Go's own reports (`burlerengine/fix.go:195`): survives, since D1 covers it.
  - `repairReportBeforeGate`'s rewrite prompt (`burlerengine/engine.go:106`): survives; the fix check re-runs on every gate attempt.
  - `fixedExit` archives the whole round on a failed gate (`burlerfix.go:320`): survives.
  - `roundComplete` checks only that files exist (`burler.go:38`): survives, because D2 adds the fix-check condition.
  - treadle and `lym burler run` hydrate fixer reports (`treadleengine/run.go:434`, `burlercli/run.go:85`): survives as today's text.
- D2, the parent's answer beside a held finding:
  - A decision file of cause `decision` is settled only by an accept, and that accept ends the segment: **breaks**.
  - The `Added after Discussion` count against `lym loom decision replace`: **breaks**.
- D3, the `prior` key and its readers:
  - `ParseCurrentReview` and `checkReviewKeys` (`verdict.go:184`) feed `acceptReview`, `acceptedReview`, `sendDecisionFindings` and `CheckReviewFile`/`validate-review`: **breaks**.
  - `ParseReview` in the Bouncer (`bouncerdecision.go:63`, `bouncer.go:1085`): survives, since its decoder tolerates the new keys.
- D3/D4, the round-check entry's inputs:
  - The acceptance record, generation record, decision files and snapshot are all private to `shedadapters`, which imports `burlerengine` (`burler.go:14`): **breaks** for an entry in `burlerengine` and for `burlercli`'s `validate-review`.
- D4, the changes file: the review producer's archive list (`burlerreview.go:284`) never names it, and a re-run rebuilds it: survives.
- D5, the snapshot record:
  - Review-producer archive exits (`burlerreview.go:284`): survives; not in the list.
  - A fix-gate failure archive (`burlerfix.go:128`): survives.
  - A new generation's `archiveRunDir` and `CarryRecords` (`archive.go:55`): it is not carried, which is correct because the round is full.
- D6, stalemate settle and replay:
  - `Call` → `standsAsRecorded` → `settle(…, false)` → `settleUndecided` (`bouncer.go:504`): survives; the recorded cause re-escalates.
  - The auto-accept's `!escalated` guard on replay (`bouncer.go:1355`): survives.
  - `retireContinue` archiving an escalation record at the hard cap (`bouncer.go:1284`): survives, since a hard-cap record replaces it.
  - `routeDecisions` takes decision-class rounds before `settle` or `settleUndecided` run (`bouncer.go:1038`): **breaks**; no post-fix id is recorded for those rounds.
  - `escalationReason` and the brief render the cause generically (`bouncer.go:1590`): survives.
  - `RecordCirclingDecision` → `valid()` (`circling.go:72`, `escalation.go:31`): survives, since D6 adds the value.
  - `arbiterstop.Eligibility` (`arbiterstop.go:113`): survives, since D6 names it.

**Findings**

1. **BLOCKING · design** · `decision-record.md` D2, second paragraph, and D3's answered-decision bullet.
   - Summary: half of D2's answer channel can never fire. When round N runs, round N-1 never holds a settled decision file of cause `decision`.
   - Only an accept can be settled (`circling.go:232`).
   - An admitted decision accept runs `acceptRound`, which settles and returns `Done` (`bouncerdecision.go:174`, `bouncer.go:1313-1330`), so the segment ends and no round N follows.
   - A new generation archives the run directory (`archive.go:55`).
   - A `continue` is never settled. A refused accept is retired by `retireContinue`, which moves the file aside (`bouncerdecision.go:178`).
   - So the only live answer source is the added-entry count, and the D2 acceptance criterion about decision files tests an unreachable state.
   - Fix: drop the decision-file source from D2 and its acceptance criterion, and state that a parent's answer reaches round N only as a `lym loom decision add` entry.

2. **MEDIUM · design** · D2 and D5 (the count of `Added after Discussion` entries).
   - Summary: `lym loom decision replace` breaks the count. Replace removes added entries when it folds them (`loom-discussion-spec.md:134-137`).
   - Counterexample: round N-1's snapshot holds 2. The parent folds both into the main decisions with replace, then adds 1 answer. The record now holds 1 added entry, which is not past 2, so the new answer is never rendered and the finding shows as still with the parent.
   - The existing decision snapshot compares titles for this reason (`bouncerdecision.go:292-335`, `addedSinceSnapshot`).
   - Fix: D5 records the titles of the added entries, as `writeDecisionSnapshot` does, and D2 renders the entries whose titles the snapshot lacks.

3. **MEDIUM · design** · D3 (`ParseCurrentReview keeps the shape checks alone`).
   - Summary: the existing shape check refuses the new top-level `prior` key.
   - `checkReviewKeys` refuses every top-level key except `verdict` and `findings` (`verdict.go:184-200`).
   - That makes every strict reader refuse an incremental review: `acceptReview` (`handoff.go:55`), Fix's `acceptedReview` (`fix.go:217`), `sendDecisionFindings` (`burlerfix.go:90`) and `validate-review`/`CheckReviewFile` (`verdict.go:232`).
   - D3's promise that a review passing on the cap is accepted with its findings then fails.
   - Fix: D3 states that the shape check admits `prior` as a third top-level key, and that the round-check entry alone judges its presence.

4. **MEDIUM · design** · D3 (round-check entry in `internal/burlerengine/verdict.go`, "given the run directory and round number") and `validate-review`.
   - Summary: from those two inputs the entry cannot reach the facts it must judge.
   - To know whether round N is incremental and whether a decision-class finding was answered, it needs:
     - round N-1's acceptance record (`burler.go:45`, `readAcceptance`);
     - the generation record's Amendment mark (`amendment.go:83`, `readGenerationRecord`);
     - the decision snapshot.
   - All of these are unexported `shedadapters` state, and `shedadapters` imports `burlerengine` (`burler.go:14`). Reading them from `burlerengine` is an import cycle or a second parser of `shedadapters`' files.
   - Fix: the review producer computes a round context (incremental flag, prior findings with answered marks, touched paths, parent-ask ids) and hands it to `burlerengine`. The entry judges against that context, and `validate-review` rebuilds it through an exported `shedadapters` builder.

5. **MEDIUM · design** · D6, first paragraph ("The first time the Bouncer settles round N, it computes the target id…").
   - Summary: a round that `routeDecisions` takes never reaches `settle` or `settleUndecided`, so it records no post-fix id.
   - `settleUnlessRefused` returns from `routeDecisions` before `settle` (`bouncer.go:1038-1040`). A decision `continue` is settled by `settleDecisionContinue` (`bouncerdecision.go:149-161`).
   - D6 covers a missing snapshot record, but not a snapshot with no post-fix id.
   - If a later round's count computes that id lazily, it hashes the tree after later fixes and misreads the round.
   - Fix: record the post-fix id at the first settle entry for every judged round, before `routeDecisions`. Or state that a round with no recorded post-fix id counts as changed.

Tool calls:
1. grep for other fixer-report readers and `repairReportBeforeGate`: the manifest lacked the callers outside `burlerengine` and `shedadapters`.
2. `archiveRunDir`/`CarryRecords` and escalation-cause readers: the manifest lacked the new-generation archive path.
3. The escalation render per cause and the arbiter stencil's cause list: the manifest lacked `escalationReason` and the arbiter stencil.
