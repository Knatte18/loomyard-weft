# umbrella-anchor run stopped at batch 06

Batches 01 (lyxcwd-own-marker) and 03 (fabric-target-locations) finished with status done.
Batch 06 (clone-records-marker-on-prime-weft) is failed.
Batches 08, 11, 13, 15 and 18 were not begun.

`recover-batch 6` refused with `needs_fresh`: the audit flagged the fork's `lyx gate test --card 6-clone-records-marker-on-prime-weft` commands as fabric references.
The parent, `ly:orch`, judged the findings false: the refscanner's bare-word rule flags any word ending in `-weft` (#874).
`lyx webster accept-audit --batch 6` refused twice, because the fork's commands were not all read-only and the batch also committed card 7's work.
Its only way forward is `lyx webster reset --to batch-start --batch 06`, which the parent runs once this run has stopped and then resumes the run.

No batch digest for 06 was read, so there are no deviations, departures or renames to report, and no scope guard result for it.
