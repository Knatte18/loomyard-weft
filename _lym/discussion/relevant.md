# Relevant files

- `internal/shuttleengine/claudeengine/capabilities.go` `Claude.BoundaryBlock`, `Claude.RewakeOutput`, `Claude.HookInput`, `stopBlockReason` — renders the Stop and tool-boundary hook output and the rewake that D1, D2 and D3 change
- `internal/shuttleengine/claudeengine/settings.go` `hookCommand`, `creelStopHookCommand`, `creelWaitHookCommand`, `buildSettings` — the hook entries; the waiter entry gains D4's keys
- `internal/creelcli/hookwait.go` `hookWait` — writes the bell line on a ring today; writes and acknowledges the claimed lines under D3
- `internal/creelcli/hookread.go` — the claim client for tool-boundary and Stop hooks, its exit codes and acknowledgement
- `internal/creelengine/verbs.go` `ClaimAtBoundary`, `HookWait`, `ringHook`, `hookWakeState`, `hookFieldCap`, `boundaryClaimBudget`, `stopBlockCap` — the claim rules and the ring the D3 claim joins, and the claim sites D5 logs
- `internal/creelengine/deliver.go` `BellLine` — the bell line, no longer the ring's output
- `internal/creelengine/doc.go` — states the ring's bound and the hook delivery rules the docs update
- `internal/creelcli/doc.go` — states hook-read's rendering and exit codes
- `internal/creelcli/smoke_llm_test.go` `TestSmokeCreelHookDelivery` — the tier3 real-session subtests the acceptance criteria extend
