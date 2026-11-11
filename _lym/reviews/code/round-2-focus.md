---
round: 2
focus:
  - "internal/adoptcli/worker.go runWorker: when a resume restarts at the reconcile step, do the end envelope and the stored record still hold the pairs mutations and the pause step's slug fields from the earlier process? Check that record.Steps and record.StoppedPart are cleared only where a fresh record or a restart requires it."
  - "internal/adoptcli/steps.go stepDone: does the way-forward command the stop names (the --keep-local and --drop-local variants) pass Request.SameSubjects in judgeStanding against the stopped record, so it resumes rather than being refused as other subjects? Check beamcli/adoptsection.go prints the same command."
  - "internal/adoptcli/steprun.go recordStep: is the clearing of a stored StoppedPart on a later success pinned by a test in internal/adoptcli/worker_test.go, including that the end envelope holds the step's part once?"
---
