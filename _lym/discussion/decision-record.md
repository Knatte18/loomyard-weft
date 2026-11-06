## Goal

```markdown
Imported from [issue #73](https://github.com/Knatte18/loomyard/issues/73).

Follow-up to #72, with what is known now. #72's workaround does not work.

Seen: every drift judge in the Models hub halts at Drift-Verify or Drift-Recheck with 'decision halt: the judge session changed what it may not: records: src/csharp/NORCE.Models/_lym/usage/session-<uuid>.json was added'. Sometimes '_lym/drift/judgement-<stamp>.yaml was added' is named too. Three runs are blocked now: cuttings-fitted-rheology (Drift-Recheck), api-density-coefficients (Drift-Verify) and baseline-provenance-outside (its judge will hit it).

Cause: internal/driftengine/judgesnapshot.go, Changes, exempts only judgement.yaml and the loomRecordsDirs entries (internal/loomcli/driftwiring.go loomRecordsDirs: the run directory under _lym/shed and _lym/loom/friction). Every judge session writes lym's own usage record under shedrun.UsageDirRel() (_lym/usage/session-<id>.json), which is not exempt.

No way out: deleting the usage record and running 'lym loom resume' re-captures and re-judges with a new session, which writes a new usage record and halts again. No lym verb gets a run past it.

Expected: the judge's snapshot check exempts lym's own records: add the anchored shedrun.UsageDirRel() to loomRecordsDirs, and exempt the judgement-<stamp>.yaml copies beside judgement.yaml. A judge that changed nothing never halts.


## From issue #72

Imported from [issue #72](https://github.com/Knatte18/loomyard/issues/72).

Seen: task run cuttings-fitted-rheology blocked at Drift-Recheck right after the drift judge recorded its rulings, with 'decision halt: the judge session changed what it may not: records: src/csharp/NORCE.Models/_lym/usage/session-<uuid>.json was added'. That file is lym's own usage record of the judge session, not a change the judge made.

Workaround: the parent deleted the usage record by its path and ran 'lym loom resume'.

Expected: the judge's snapshot check excludes the usage records lym writes for its own sessions (_lym/usage/), so a judge that changed nothing never halts.

### Issue comment, 2026-11-06

Also seen on a second judge run of the same task (baseline-provenance-outside, Drift-Verify): besides the new usage record, the guard flagged `_lym/drift/judgement-20261106T083132Z.yaml` as added. That is the previous judge's `judgement.yaml`, kept under a timestamped name when the next judge ran, so lym's own rotation of the judgement file trips the guard too. Workaround: deleted both added paths and ran `lym loom resume`. Expected: the snapshot check excludes lym's own records under `_lym/drift/` and `_lym/usage/`.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
