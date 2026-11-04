---
round: 1
focus:
  - "internal/shuttleengine/sessionstate.go, `SessionFold.settle`: with unproven liveness no longer settling, does every other path still rank correctly (dead process, unreadable events file, empty signal history above the derived state), and can Claude or another engine whose liveness is unproven for a transient reason now read a busy or idle state it cannot back?"
  - "Callers of liveness and the pane fallback (readiness, conditioned send, wait loop): does any caller still treat `LivenessUnproven` as a reason to fall back to the pane or to log a disagreement, and does the new ordering change a decision they made on an unknown state?"
  - "internal/shuttleengine/spec.go and internal/shuttlecli/run.go: the hidden-anchor refusal comes from two places; does each reject before any strand exists, do both name why and own-window as the way forward, and is any other entry path (resume, other CLI verbs, spec loaded from file) left able to start a hidden strand?"
  - "Decision record item 1 allows either a working hidden strand or a refusal naming why; confirm the refusal's stated cause matches what reed actually does for a hidden placement, and that no doc or help text still advertises `--anchor hidden` for shuttle runs."
  - "Tests changed in fakes_test.go, gate_test.go, run_test.go, session_test.go, conditionedsend_test.go: the diff rewrites tests that forced the pane fallback through unproven liveness; check each rewrite still exercises the behavior its name claims, and that the new tests for the changed `settle` order and the refusal cover each new behavior without duplicating existing coverage."
  - "internal/shuttleengine/doc.go and internal/shuttleengine/codexengine/doc.go: read the changed doc text against the code as the diff leaves it, and against the repository's comment conventions."
---

Seed focus for a diff touching two faults: the hidden-anchor refusal and the Codex session-state fold.
