# Card 12 — delete-burler-shuttle

**Delete:**
- `internal/burlerengine#Shuttle`
- `internal/burlerengine#runnerShuttle`
- `internal/burlerengine#RunnerShuttle`
- `internal/burlerengine#runnerShuttle.StartGated`
- `internal/burlerengine#runnerShuttle.ProbeGated`
- `internal/burlerengine/shuttle_test.go`

**Intent:**
Delete burlerengine's own `Shuttle` seam, its `runnerShuttle` adapter and `RunnerShuttle` (decision D1), now that card 8 has every caller of `burlerengine.New` pass a `seatengine.Shuttle`.
`shuttle_test.go` goes with `RunnerShuttle`, whose only coverage it is; `seatengine.RunnerShuttle` keeps its own.
`Handle` stays in `shuttle.go` as the type of `LiveHalf.Handle`, and the file's header comment narrows to it.
It is a pure deletion and relies on the existing tests.

**ImpactSummary:** Deletes burlerengine's Shuttle seam and its adapter in shuttle.go plus their test; no caller remains after card 8.
