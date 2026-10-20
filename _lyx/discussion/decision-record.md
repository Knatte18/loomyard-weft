# Decision record — watchdog-reap-test-xdr

## Goal

`TestWatchdogReap`'s steps stop failing on a slow host, such as one running Cortex XDR, where each new tmux and git process image is held before it runs.

## Scope

- `internal/reedcli/watchdogreap_integration_test.go`: the waits for a reap, or a discovery, that the step expects to succeed.
- No production code changes: `compressedReapTiming` keeps its compressed cadence and the production confirmation count.

## Decisions

### A fixed success-path bound replaces the cycle-derived window

Decision: every `waitForCondition` in `TestWatchdogReap` that waits for an outcome the step expects, an orphan session gone or a fresh session listed, waits one named constant of several tens of seconds instead of `DiscoveryCycle*OrphanGoneCycles*20` or `DiscoveryCycle*10`.
Rationale: each discovery cycle spawns tmux and git, so a host that holds new process images stretches every cycle far past the compressed 30 ms cadence, and a window derived from that cadence cannot tolerate it.
`waitForCondition` polls, so a fast host returns as soon as the condition holds and never pays the bound.
Bound: the constant only lengthens how long a broken reap takes to fail; it lets no wrong outcome pass.

### Short checks that assert time running out stay short

Decision: the "still live before `OrphanGoneCycles` cycles" check in `EndToEndAndSiblingSurvives` and the cancel deadline in `LoopStaysLiveDuringReap` keep their bounds.
Rationale: a stall makes the first pass rather than fail, and the second bounds an in-process select with no spawn on its path.

## Constraints

- `PATTERN-test-speed`: no test waits out a production duration; the compressed timing stays, and the new bound is a deadline polled against, never slept.
- The test keeps proving the consecutive-cycle rule, so `OrphanGoneCycles` stays at its production value.

## Auto-mode assumptions

- The process-image holding the issue reports also stretches the setup the step ran for 11 s; the setup has no window of its own to widen.
- Thirty seconds covers the slowest reap seen on such a host with margin, since the failing step's whole run took under 12 s.

## Open risks

- A host slower than the bound still fails; the bound is a named constant, so raising it is a one-line change.

## Acceptance criteria

- `TestWatchdogReap` passes under `lyx gate test --tags tmux ./internal/reedcli`.
- No wait in the file that expects success is derived from `DiscoveryCycle`.
- The early-reap check still fails a reap that fires on the first observation.
