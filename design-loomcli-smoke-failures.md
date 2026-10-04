# loomcli smoke tier: five bootstrap/strand tests fail on main

`loomcli-smoke-failures` · Note · bug, testing

`go test -tags smoke ./internal/loomcli/` fails five tests on `main` (seen 2026-10-03 at `6a74ee1be`, during the PR-Gate review of #347, which fails the same five).
Each expects one `loom-status` or driver strand and finds none:

- `TestSmokeBootstrap_BringsUpSessionStrandAndDriver` — `smoke_test.go:530`: status strands named "loom-status" = 0; want exactly 1
- `TestSmokeBootstrap_SecondInvocationDoesNotSpawnASecondDriver` — `smoke_test.go:592`: status strands named "loom-status" after two bootstraps = 0; want exactly 1
- `TestSmokeDriverStrand_ReentrantAcrossThreeBootstraps` — `smoke_driverstrand_test.go:216`: driver strands after the first bootstrap = 0; want exactly 1
- `TestSmokeStep_LeavesAPreexistingStatusStrandUntouched` — `smoke_drivingsurface_test.go:100`: status strand guids after step = []; want exactly the one added before the step
- `TestSmokeStart_LLMRunRemovesEveryStatusStrand`

Either the smoke tests are stale against the current bootstrap, or bootstrap no longer brings up the status and driver strands it should.
Nothing runs the smoke tier in the normal verify, so the failures went unnoticed.


From GitHub #348 (intake 2026-10-04).
