---
round: 1
focus:
  - "Goal: the diff must settle which message the died-driver case surfaces (loop-exit reason or the poisoned status file's invalid state) and make code or test match; check the choice is applied consistently in `internal/loomcli` `TestSmokeBootstrapLifecycle/died_driver_proceeds_to_the_success_envelope_and_logs_why` and in the shed or driver code it reads."
  - "Goal: the reason the run halted must stay in the driver log; check the diff does not drop or weaken that log line, and that a failure path still reports it."
  - "Scope: the board entry names one wording mismatch from `d1535412` (shedverbs loop-exit recording); check the diff does not widen into unrelated shed, loom or driver behavior, and does not revert `d1535412`'s recorded exit reason for other callers."
  - "Test economy: if the test assertion changed, check it still pins what the case is for, rather than loosening to a substring that any message satisfies."
---

The seed pass found an empty review range (HEAD equals the base), so the focus follows the board entry's Goal alone.
