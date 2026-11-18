```yaml
lens: way-forward
outcome: reported
```

Lens: way-forward. I checked each failure and recovery end that cards 1 to 5 add, and whether the next step it names can succeed.

Each refusal, its next step, and whether that step progresses:
- **Card 3, failed move, case 1 (no runs strand):** the next step is `lym loom start` in the beam's worktree. It progresses: after the fallback the runs strand holds the overview, so `resolveStatusStrandAction` answers add or replace, and card 2's `statusStrandAddSpec` builds the beside spec.
- **Card 3, failed checklist sidecar write:** the next step is `lym loom start`, which replaces the strand once. It progresses: the sidecar mismatch gives `statusStrandReplace` with the beside spec.
- **Card 3, failed runs-strand add after a move:** the next step is `lym beam start`. It progresses: there is no runs strand and `loom-status` is present, so case 1 moves it again and then adds the runs strand. This covers the concurrent `lym loom start` case too.
- **Card 3, failed move, case 2 (current runs strand, dead `loom-status`):** the next step is `lym loom start`, which replaces the dead strand beside the runs strand. It progresses, but see findings 1 and 2.
- **Card 3, failed re-read after a failed move:** the next step is "falls back to the add", whose failure names `lym beam start`. It may not progress: see finding 3.

Findings:

1. **MEDIUM — D2 failures, card 3 Intent step 2.** Card 3 does not say whether "After a failed move it reads the strands again: a `loom-status` strand still present is replaced by the runs strand" applies only when there is no runs strand. Read literally, a failed move in case 2 (runs strand current, `loom-status` dead) replaces `loom-status` with a second runs-named overview spec. Reed refuses that, through the role clash in `validateNaming` or the one-overview rule in `internal/reedengine#Engine.ReplaceStrand`, so the report gains a spurious runs-strand warning naming `lym beam start`. D2's failure bullet places the re-read after the case-1 move, and no row covers a failed move in case 2.
   - Evidence: card 3 Intent items 1 and 2; `internal/reedengine/replace.go` `ReplaceStrand` doc ("replacing another strand with an overview strand is refused only while a third strand holds the overview").
   - Fix: scope the re-read and fallback to "with no runs strand". Add a row: a current runs strand plus a dead `loom-status` whose replace fails wants one warning naming `lym loom start` and no runs-strand replace or add.

2. **LOW — case-2 warning text, card 3 `moveChecklist`.** The failed-replace warning says "could not move the loom status strand out of the overview window". In case 2 the strand is a dead one beside the runs strand and was never in the overview window, so the warning misstates the state the operator acts on. The way forward it names still progresses.
   - Evidence: card 3, the `checklistWarning("could not move the loom status strand out of the overview window, …")` text; D2 case 2 in `_lym/discussion/decision-record.md:39`.
   - Fix: a wording that fits both cases, such as "could not relaunch the loom status strand in a window of its own".

3. **LOW — fallback after a failed re-read, card 3 Intent step 2.** "A failed read there falls back to the add". If the move failed before removal, `loom-status` still holds the overview, so reed refuses the add (one overview strand per session). The warning then names `lym beam start`, which only progresses once the cause of the failed read clears; the card leaves this unstated and no row reaches it. The first read already gave `loomStatus.GUID`, so replacing that GUID with the runs spec ends in the same place as the add when the strand is gone (refused either way) and succeeds when it is present.
   - Evidence: card 3 Intent step 2; `internal/reedengine/replace.go` `ReplaceStrand` doc on the overview anchor.
   - Fix: after a failed re-read, replace the first read's `loom-status` GUID with the runs spec, falling back to the add only if that replace fails. Or keep the add and state why its refusal is acceptable, with a row that reaches it.

4. **LOW — the record's way forward `lym loom start` while the pawl run is live, cards 2 and 3.** The cards rely on a re-entrant `lym loom start` in the beam's worktree reaching `ensureStatusStrand` while the pawl driver is live. No card names a test or code path showing that `start` reaches `ensureStatusStrand` before any refusal over a live run, such as the merge or park checks in `internal/loomcli/start.go`. The record's Scope assumes it, so this is noted, not proven broken.
   - Evidence: `internal/loomcli/start.go:151-225` holds refusals that name `lym loom start` as their retry; the order against `ensureStatusStrand` was not confirmed within the call budget.

Tool calls made:
- grep of `internal/loomcli/start.go` for `ensureStatusStrand`, the refusals and the returns, to see whether `lym loom start` reaches the status strand on a live run; my context did not hold `start.go`.
