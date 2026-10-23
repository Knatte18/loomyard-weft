---
round: 1
ledger:
  - key: failuretrace-newtracer-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: failuretrace-doc-derives-no-paths-claim
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised two non-gating findings.
F1 (`failuretrace-newtracer-untested`) asks for a `NewTracer` test over a missing plan directory and a missing Webster state.
F2 (`failuretrace-doc-derives-no-paths-claim`) asks for `internal/failuretrace/doc.go` to say the tracer is told the worktree root and anchor path and takes other paths from the owning packages.
