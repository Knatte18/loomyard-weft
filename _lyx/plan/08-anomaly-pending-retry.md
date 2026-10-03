# Card 8 — anomaly-pending-retry

**Edit:**
- `internal/loomcli/selfreport.go`
- `internal/loomcli/selfreport_test.go`

**Uses:**
- `internal/loomengine#DetectAnomalies`
- `internal/loomengine#RenderAnomalyBody`

**Intent:**
A detected anomaly whose filing call fails is still filed after the run is resumed past it, closing the one way a halt is lost once card 7 runs detection at every step (the hub's "Friction on halt" check; evidence in the overview).

Today `runFilingPass` leaves a failed title unrecorded so that it is "retried next run", but the next pass re-detects only from the then-final status: a `blocked` or `failed` halt that is resumed and goes on is never detected again, and its anomaly is never filed.

`selfreportFiledMarker` gains `Pending []pendingAnomaly`, where `pendingAnomaly` holds the title and the rendered body of an anomaly whose filing failed.
`runFilingPass`, per call:

1. retries every pending entry whose title is not yet recorded, recording the title and dropping the entry on success;
2. files the newly detected anomalies as today; a failed call appends the anomaly to `Pending` unless its title is already pending.

The marker is written after each change, as today, so a crash between calls loses at most the one in-flight change.
A marker written before this card decodes with an empty `Pending`.
The pass now also runs when detection finds nothing new but `Pending` is non-empty.

What this lets through, and the bound: a retried issue is filed later than the halt it describes; each title is still filed at most once, and pending entries are retried only from the filing pass, which keeps every existing skip (knob off, busy, cancelled context).

`selfreport_test.go` gains:

- a halt anomaly whose filing fails is recorded as pending, and a later pass whose final status is `running` (the halt resumed) files it with the original title and body;
- a pending entry that fails again stays pending and is not duplicated;
- a pending title already recorded as filed is dropped without a filing call;
- a marker JSON without the `pending` key reads as empty pending.

**ImpactSummary:** `internal/loomcli`'s filing pass and its machine-local marker format; the marker change is additive and read by nothing outside this file.
