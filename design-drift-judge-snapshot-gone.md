# drift: a judge snapshot gone between resumes halts the run for a hand check

`drift-judge-snapshot-gone` · Note · bug, drift, loom

Imported from [issue #76](https://github.com/Knatte18/loomyard/issues/76).

What happened: Drift-Verify halted blocked with 'the snapshot of both checkouts the judge was spawned under is gone, so what its session changed cannot be told'. The snapshot lives under the hub's _bolt/.lym/drift-judge/<task> directory, which earlier halts in the same run pointed to as the restore source.

Why abnormal: the snapshot disappeared between consecutive resumes of the same run, likely removed by the operator's or parent's cleanup step ('deletes ... the changed scratch directory' in the way-forward text of the earlier halts) while the next judge was already spawning, or by lym itself. A resume then has no baseline and halts again, forcing a manual check of both checkouts.

What to do: make the way-forward cleanup and the snapshot lifecycle not race with a resume; take a fresh snapshot automatically on resume when the old one is gone instead of halting, as the halt text itself says resume would do.

Provenance: task api-density-coefficients; note loom-halt-5.md.