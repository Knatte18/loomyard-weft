---
round: 1
ledger:
  - key: anchor-restart-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: beamupworktree-location-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: orchcli-doc-says-alone
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised three findings, all open.
`anchor-restart-untested` (F1) gates: the restart loop and `beamStart` in `internal/fabriccli/anchordeps.go` have no test.
`beamupworktree-location-unpinned` (F2) asks for the prime-versus-beam location choice in `internal/orchcli/start.go` to be pinned.
`orchcli-doc-says-alone` (F3) asks for `internal/orchcli/doc.go` to drop the word alone.
