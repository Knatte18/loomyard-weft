---
round: 1
focus:
  - "internal/battencli/startguard.go and cleanup.go: does a raddle start clear only the done motif run under the same slug, and leave any other run's directory, a live run and a non-done run untouched? Settle what happens when the clear fails or the directory is already gone."
  - "internal/fabricengine/replay.go with originreconcileended_integration_test.go: does a raddle run at Raddle-File stop being ended by its own entry's done mark, while a run whose origin really ended still reconciles as ended? Settle whether the new condition can mask a genuine ended origin."
  - "internal/battencli/dobbyreads.go and cli.go: does the `child_done_batten_halted` hint now agree with what the run does, so status and the Run-Shed refusal no longer contradict each other?"
  - "internal/loomcli/wiring.go `sketchFinish`: when the board refuses the Sketch, is the parent mailed with the refusal, the dependent entries and both ways forward (drop or move dependencies, or set recipe raddle)? Settle the failure path when the mail itself cannot be sent, a run with no parent, and that the draft-task fallback and Warn are kept."
  - "Scope against the decision record's three notes: is each of raddle-same-slug-completed, raddle-end-fails-batten and motif-sketch-refusal-silent carried by the diff, and does the diff change anything outside them?"
  - "Test economy across startguard_test.go, commitstatus_integration_test.go and the replay tests: does each new test cover behavior the others do not, and does any new behavior lack coverage?"
  - "Doc comments in internal/fabricengine/doc.go, internal/loomcli/doc.go and the changed godocs: do they follow the repository's own comment conventions, and do the docs the repository's rules name for these changes land in the diff?"
---

Seed focus from the decision record's three notes and the diff stat of the three commits in the range.
