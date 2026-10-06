# Test suite prune: stopped at batch 10

Batches 01 to 09 are done and committed: the run-alone flag form, out-of-process exclusion, the keep directive, report splicing, the before measurement, the redundancy rerun, the shedadapters and shuttleengine prunes, and the websterengine untag.

Batch 10 (websterengine-prune) is `dead`: a previous session's recovery of it already failed terminally.
This session ran no batch.
It only read `lyx webster status`, which confirms batch 10 as a terminal `dead` recovery batch.

Batches 11 to 17 are not started: reedengine, loomcli and fabricengine untag and prune, and the after measurement.
Batch 11 assumes batch 10 is committed, so none of them can begin until an operator looks at batch 10.
