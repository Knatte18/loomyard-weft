# drift: the judge guard halts on lym's own scope.json write

`drift-judge-scope-record-halt` · Note · bug, drift, loom

Imported from [issue #74](https://github.com/Knatte18/loomyard/issues/74).

Seen: after a judge snapshot was gone, 'lym loom resume' halted again at once: 'the judge session changed what it may not: records: src/csharp/NORCE.Models/_lym/drift/scope.json was changed'. The only change to scope.json is lym's own removal of "snapshot_lost": true.

Cause: internal/driftengine/verify.go, resumeJudgeSession, takes the new snapshot with TakeJudgeSnapshot first and then clears scope.SnapshotLost with WriteRecords. That record write lands after the snapshot, so judgeSessionChanged names scope.json as a change the judge made. The designed way to recover a lost snapshot therefore always halts.

Workaround used: set the snapshot's records head in snapshot.json to the records checkout's current HEAD, then 'lym loom resume'.

Expected: clear SnapshotLost before taking the snapshot, or exempt the drift records lym writes itself (scope.json, judged.json) from the judge's write guard. Related: #72, #73.


## From issue #75

Imported from [issue #75](https://github.com/Knatte18/loomyard/issues/75).

What happened: the Drift-Verify step halted blocked four times in a row (anomaly escalation-to-human) with 'the judge session changed what it may not: records: ...'. The paths named were lym's own bookkeeping: a per-session file under _lym/usage/ (reported once as added, twice as changed) and _lym/drift/scope.json (changed). The judge session writes these through lym's own machinery (usage accounting, drift scope), not by editing task files.

Why abnormal: the guard treats lym-owned record files under _lym/ as forbidden judge edits, so a normal judge run cannot pass. Each halt needs the operator to restore or delete the paths and run lym loom resume, and the same halt recurs on the next judge run.

What to do: exclude lym-written record paths (_lym/usage/, _lym/drift/) from the judge-change comparison, or snapshot and compare them in a way that ignores lym's own writes. Also consider not halting for paths lym itself touched during the judge session.

Provenance: task api-density-coefficients; notes loom-halt-2.md, loom-halt-3.md, loom-halt-4.md, loom-halt-6.md.