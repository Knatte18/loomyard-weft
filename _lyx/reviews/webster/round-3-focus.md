---
round: 3
focus:
  - "internal/websterengine/cardgate_test.go: does every top-level test the range adds or rewrites, and each of its subtests, call t.Parallel or carry a comment at the test or atop the file naming the process-global state it touches?"
  - "internal/gatecli/run.go and internal/gateslot/config.go: for an absent gate.yaml, does the CLI envelope name one way forward that agrees with the `## gate` row in contracts/specs/refusal-spec.md, and does a gatecli test reach that trigger?"
---

The range's fix commits for these two areas are new text; read them as part of the diff.
