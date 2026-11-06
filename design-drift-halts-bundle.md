# Drift: lym record writes, a gone snapshot, capture provenance and a retired placeholder halt runs

`drift-halts-bundle` · Task · bug, drift, loom, config, reconcile · running · Describe

Four bugs that halt drifting runs in the Models hub (`cuttings-fitted-rheology`, `api-density-coefficients`, `baseline-provenance-outside`).
`98bf70b0` fixed the judge guard for `_lym/usage/` and the archived `judgement-<timestamp>.yaml` only; the issues predate it or report the same runs.
Check each case against today's `internal/driftengine` before changing it.

## 1. The judge guard halts on `_lym/drift/scope.json` (issues 74, 75, 77)

The guard reports `the judge session changed what it may not: records: .../_lym/drift/scope.json was changed`.
The change is lym's own: `resumeJudgeSession` in `internal/driftengine/verify.go` takes the new snapshot with `TakeJudgeSnapshot` and then clears `scope.SnapshotLost` with `WriteRecords`, so `judgeSessionChanged` names that write as the judge's.
The designed recovery of a lost snapshot therefore always halts.

Fixed when no record lym writes itself during or around a judge session reads as a judge's edit, `scope.json` and `judged.json` included, and a resumed judge does not trip on them.
Fix the order in `resumeJudgeSession`, and exempt what lym writes, so the next such record does not cause the next halt.
A judge that edits a task file or a record it may not touch must still halt.

## 2. A gone snapshot halts the run (issues 76, 78)

Drift-Verify and Drift-Recheck halt with `the snapshot of both checkouts the judge was spawned under is gone, so what its session changed cannot be told`, and the way forward asks for a hand check of both checkouts.
The snapshot lives under `_bolt/.lym/drift-judge/<task>`.
It went because the neighbouring guard halt's way forward tells the reader to delete `the changed scratch directory`.

- No halt text tells anyone to delete the snapshot directory, or the recovery it names keeps the snapshot a resume compares against.
- A resume with the snapshot gone takes a fresh one and goes on, as the halt text itself says resume would do, when no judge session is live that the lost snapshot was guarding; where a live session's changes cannot be told, keep the halt and say so.
- The snapshot's lifecycle does not race a resume.

## 3. Capture provenance reads as drift (issues 71, 79)

Drift-Recheck blocks after `compare_passed: true` with `the code changed what the expectation does not cover: changed in case "_source.txt" at key "commit" ... at key "captured_at"`.
The Models hub's baseline harness writes `_source.txt` beside the captured cases, so it differs in every capture of a new commit, and every run with an empty expectation halts; each needs a `lym drift amend`.

The file belongs to the hub's harness, so lym hard-codes no name: add a hub-wide list in `drift.yaml` of volatile capture files or keys that the diff ignores, documented in the template.
Report in the change description what the Models hub must set.

## 4. A retired `{skip_file}` stays in hub config and blocks runs (issues 69, 70, 79)

A hub's `drift.yaml` with `-e NORCE_BASELINE_SKIP_FILE={skip_file}` in `capture.command` and `compare.command` blocks at Drift-Recheck with `the drift current capture command holds {skip_file}, which drift no longer renders`, and `lym config reconcile --layer default --apply` does not remove it.

- Reconcile migrates the retired placeholder out of existing drift commands, the whole `-e NAME={skip_file}` argument with it.
- A leftover placeholder is found at config load or preflight, before a run starts, rather than at Drift-Recheck.
- The error names a way forward that works from the session that gets it; a beam orch is refused hub-wide config writes.
