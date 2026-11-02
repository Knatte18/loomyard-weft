---
round: 3
focus:
  - "Where the wrapped gate closure may spawn a fixer: the wait loop's finalize runs the gate evaluation (`internal/shuttleengine/wait.go`, `evaluateGate(true)` before the writer's strand is removed) and a Gate closure takes no argument (`gate.go`). Settle whether the record states that a fixer is spawned only at an evaluation reached at the writer's idle turn boundary, how the wrap knows which kind of evaluation it is in, and that the deadline overrun bound and the finalize acceptance criterion match."
  - "The identity that keys a row visit for the fixer record set: the run scratch directory is per run (`shedrun.ScratchDir`). Settle whether the record names the identity and where it is read, and states whether a crash re-step of the same row keeps the set, so the Expired probe and wait-only entries see a live fixer."
  - "What stops a live fixer when the driver dies and the operator moves the run: `internal/shedverbs/goto.go` calls no runproc end. Settle whether the record names which verbs end the run's fixer role and what guarantees a goto or reset past a row with a live fixer leaves none running."
  - "Which module owns the set of fixer roles the gate verb's hand-off guard tests, against the decision that the gate verb learns nothing of webster; and the card fixer's runproc owner and parent directive when spawned by the detached runner (`internal/gatecli/detachfront.go`)."
  - "For the new `fixer` field on gate.yaml keys: settle whether the record states what it lets through on keys other than card and what bounds it, consistent with the Scope Out line on the darn writer."
---

Focus targets the places where the design's premise that the writer sits idle while the fixer edits may not hold, and the identity and ownership gaps around the fixer record set.
