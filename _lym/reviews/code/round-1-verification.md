---
round: 1
entries:
  - key: beam-member-routing-still-nearest-live
    status: closed
    finding: F1
    reason: "9380a4675 and 83d5cc78c: orchroute.Destination, behind FromPair, Asker, orchcli and the dobby, keeps mail at the beam orch while reed holds its strand whatever the probe says; a stopped orch (strand removed by stopOrch) passes it up; notify_integration_test.go covers a member with the probe false, and the summary bullet is replaced."
  - key: hubgeom-doc-go-stale-guard-and-consumer
    status: open
    reason: "Maps to F10 (internal/hubgeom/doc.go), which is not among the BLOCKING findings listed for this verification (only F1), so it stays open unverified."
---
