# drift: every judge halts on lym's own usage record, and resume loops

`drift-judge-usage-record-halt` · Note · bug, drift, loom

Imported from [issue #73](https://github.com/Knatte18/loomyard/issues/73).

Follow-up to #72, with what is known now. #72's workaround does not work.

Seen: every drift judge in the Models hub halts at Drift-Verify or Drift-Recheck with 'decision halt: the judge session changed what it may not: records: src/csharp/NORCE.Models/_lym/usage/session-<uuid>.json was added'. Sometimes '_lym/drift/judgement-<stamp>.yaml was added' is named too. Three runs are blocked now: cuttings-fitted-rheology (Drift-Recheck), api-density-coefficients (Drift-Verify) and baseline-provenance-outside (its judge will hit it).

Cause: internal/driftengine/judgesnapshot.go, Changes, exempts only judgement.yaml and the loomRecordsDirs entries (internal/loomcli/driftwiring.go loomRecordsDirs: the run directory under _lym/shed and _lym/loom/friction). Every judge session writes lym's own usage record under shedrun.UsageDirRel() (_lym/usage/session-<id>.json), which is not exempt.

No way out: deleting the usage record and running 'lym loom resume' re-captures and re-judges with a new session, which writes a new usage record and halts again. No lym verb gets a run past it.

Expected: the judge's snapshot check exempts lym's own records: add the anchored shedrun.UsageDirRel() to loomRecordsDirs, and exempt the judgement-<stamp>.yaml copies beside judgement.yaml. A judge that changed nothing never halts.
