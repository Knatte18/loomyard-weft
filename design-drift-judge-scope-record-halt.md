# drift: the judge guard halts on lym's own scope.json write

`drift-judge-scope-record-halt` · Note · bug, drift, loom

Imported from [issue #74](https://github.com/Knatte18/loomyard/issues/74).

Seen: after a judge snapshot was gone, 'lym loom resume' halted again at once: 'the judge session changed what it may not: records: src/csharp/NORCE.Models/_lym/drift/scope.json was changed'. The only change to scope.json is lym's own removal of "snapshot_lost": true.

Cause: internal/driftengine/verify.go, resumeJudgeSession, takes the new snapshot with TakeJudgeSnapshot first and then clears scope.SnapshotLost with WriteRecords. That record write lands after the snapshot, so judgeSessionChanged names scope.json as a change the judge made. The designed way to recover a lost snapshot therefore always halts.

Workaround used: set the snapshot's records head in snapshot.json to the records checkout's current HEAD, then 'lym loom resume'.

Expected: clear SnapshotLost before taking the snapshot, or exempt the drift records lym writes itself (scope.json, judged.json) from the judge's write guard. Related: #72, #73.
