---
round: 1
focus:
  - "internal/dobbyengine/ended.go and loop.go: does the teardown of an ended run's pair match what `lym cleanup --apply` does, and is every failure path (teardown error, partial teardown, repeated passes) handled without leaving the slot held or the pass failing?"
  - "internal/battencli/dobbyreads.go and internal/dobbyengine/starts.go: does a run whose board entry is done or gone stop counting under `max_runs` whatever batten's state says, including entries that are gone rather than done, and does a run whose entry is still live keep its slot?"
  - "internal/beamcli/dobbydeps.go: is the new teardown path safe against tearing down a pair whose entry is not actually ended, a pair with unlanded work, or one the prime still uses?"
  - "Scope against the Goal: both Idea bullets (no slot for an ended run; Dobby or reconcile tears down the pair) are met, and nothing beyond them is changed."
  - "Tests in ended_test.go, starts_test.go, dobbyreading_test.go and dobbydeps_test.go: is every added behavior covered, and does any test duplicate another's coverage?"
  - "Doc changes in internal/dobbyengine/doc.go and internal/beamcli/doc.go: do they match the behavior the diff lands, and are the repository's required docs updated for this kind of change?"
---

Seed focus from the Goal in the decision record and the diff stat; no earlier round exists.
