# drift: a judge snapshot gone between resumes halts the run for a hand check

`drift-judge-snapshot-gone` · Note · bug, drift, loom

Imported from [issue #76](https://github.com/Knatte18/loomyard/issues/76).

What happened: Drift-Verify halted blocked with 'the snapshot of both checkouts the judge was spawned under is gone, so what its session changed cannot be told'. The snapshot lives under the hub's _bolt/.lym/drift-judge/<task> directory, which earlier halts in the same run pointed to as the restore source.

Why abnormal: the snapshot disappeared between consecutive resumes of the same run, likely removed by the operator's or parent's cleanup step ('deletes ... the changed scratch directory' in the way-forward text of the earlier halts) while the next judge was already spawning, or by lym itself. A resume then has no baseline and halts again, forcing a manual check of both checkouts.

What to do: make the way-forward cleanup and the snapshot lifecycle not race with a resume; take a fresh snapshot automatically on resume when the old one is gone instead of halting, as the halt text itself says resume would do.

Provenance: task api-density-coefficients; note loom-halt-5.md.

## From issue #78

Imported from [issue #78](https://github.com/Knatte18/loomyard/issues/78).

**What happened**
Drift-Recheck halted with 'the snapshot of both checkouts the judge was spawned under is gone, so what its session changed cannot be told'. The snapshot directory under the hub's `_bolt/.lym/drift-judge/<slug>` had been removed, apparently between the previous halt and the resume, while the operator was cleaning up after earlier judge-guard halts.

**Why it is abnormal**
The way forward (check both checkouts by hand, then `lym loom resume`) puts the burden on a human to audit for edits lym can no longer detect, and the snapshot's deletion follows from the documented recovery of the neighbouring halt ('deletes ... the changed scratch directory').

**What to do**
Do not tell the operator to delete the snapshot directory in the judge-guard halt's way forward, or keep the snapshot outside the paths that recovery tells them to delete, so a resume can still compare against it.

Provenance: task cuttings-fitted-rheology; notes loom-halt-8.md (context: loom-halt-7.md, loom-halt-9.md)