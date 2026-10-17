---
round: 2
focus:
  - "internal/shuttleengine/run.go, the engine-ledger write in start before AddStrand: settle whether a failed ledger write fails Start, removes the run directory and registers no strand, and whether a test in run_test.go pins all three."
  - "internal/landingshed/config.go, migrateCoAuthoredBy: settle what an empty, whitespace-only or non-scalar co_authored_by value migrates to, and whether a TestMigrateConfig row covers it."
  - "internal/shuttleengine/run.go start, ordering of the ledger write against AddStrand: settle whether a start that fails at AddStrand leaves its engine in the ledger, and whether the code or its doc comment states the chosen behaviour."
---

Focus targets the round-1 fixes: new test and migration code that no fresh reviewer has seen.
