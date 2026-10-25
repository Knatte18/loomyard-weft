---
round: 4
focus:
  - "Resume test in `_lyx/plan/11-drift-verify-producer.md` steps 3 and 7 and the `*_at` fields in `_lyx/plan/02-drift-records.md`: settle whether the plan fixes one time format for the scope-record fields and the history times the test compares, given that `shedengine` stamps `HistoryEntry.At` and `LastAt` as second-precision RFC3339 after the producer returns."
  - "Step 7 of `_lyx/plan/11-drift-verify-producer.md`: settle when `gate_halt_at` is cleared relative to the told `verify` entry's gate, and whether a gate that ends in a context error, transient stop or interrupt leaves the next entry able to tell it must still run the gate before the compare."
  - "`_lyx/plan/14-drift-recipe-rows.md` `driftDeps` dirty-paths seam and the dirty check in step 7 of `_lyx/plan/11-drift-verify-producer.md`: settle which checkout the dirty read covers, given that `_lyx/drift/scope.json` and the logs are written by `lyx drift accept` and steps 2 and 3 before the check."
  - "Fields written to `scope.json` in `_lyx/plan/02-drift-records.md`: settle for each whether some card in the plan reads it back, and whether cards 2 and 11 state the same purpose for it."
---
