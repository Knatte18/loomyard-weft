# gate-runs-once: run stopped at batch 22

This session resumed a run whose progress trail already held ten batches as `done`: 01, 03, 04, 05, 06, 08, 13, 15, 18 and 20.
It ran no fork and no recovery.
`lyx webster status` showed batch 22 (tier-plan-docs) as a terminal `dead` recovery with `recovery_retry: false`.

The failure ladder treats that as an exhausted batch, so the run stops without beginning any later batch.
Batches 24, 27, 28, 29, 30 and 31 were never begun, and the run is `stuck`.

No digest was read in this session, so there are no deviations, departures or scope guard results to report.
The earlier sessions recorded those for the done batches.
