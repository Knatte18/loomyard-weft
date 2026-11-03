---
round: 2
focus:
  - "internal/fabriccli/anchordeps.go, `hibernateEndedPhases` and `hibernateDaemonLive`: settle whether the terminal phase values (`done`, `failed`, `cancelled`) have one source of truth with `internal/hibernateengine/request.go`'s `Phase` constants and `Terminal` method, or a test that fails when the two differ, and whether `hibernateDaemonLive` has a test covering live, dead, zero-pid and ended requests."
  - "internal/hibernatecli/deps.go `NewDobbyCarrier` and internal/beamcli/dobbywatch.go `openDobbyWatch`: settle whether the hibernate directory exists when the prime dobby's file watch opens on a hub that never hibernated, so a first request wakes the tick."
  - "internal/hibernateengine/watch.go `carry` and internal/hibernateengine/carrier.go `Tick` and goroutine end: settle whether a request found in `resuming` names the skipped wake reconcile in its notice, and whether a request submitted after a cancel is taken up once the old sequence ends without waiting for the loop timer."
  - "internal/cleanupcli/cleanup_integration_test.go near line 329: settle whether comments still name the hibernate daemon where the diff moved the docs to the request's wake."
---

The focus lists the places round 1 touched; the reviewer judges the diff afresh.
