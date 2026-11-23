# PR body's appended regression block uses internal task wording

`pr-regression-block-wording` · Task · bug, drift, landing · running · Drift-Verify

Imported from [issue #204](https://github.com/Knatte18/loomyard/issues/204).

Seen in hub mo, run hydraulics-gaps, PR NORCE-DrillingAndWells/Models#326. The 'Regression check' block that lym appends after the change description reads 'Baseline: captured at <sha>, the task's branch point.' This is internal process wording in repository-facing text, and the run's PR-Review agent cannot edit it. Expected: neutral wording such as 'the base of this branch'.