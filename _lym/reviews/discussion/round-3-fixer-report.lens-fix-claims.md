```yaml
lens: fix-claims
outcome: reported
```

Checked every claim the fix added or rewrote against the code at HEAD.

1. **MEDIUM.** Changed text: D7 step 4, "dismissing a trust or login dialog as the start path does".
   The start path dismisses no login dialog, so this states a behaviour the code does not have.
   Evidence: `internal/shuttleengine/claudeengine/startup.go:31` and `:49`. `Startup` classifies a pane only as `StartupTrustPrompt`, `StartupReady` or `StartupPending`.
   `gateAcceptNeedles` at `startup.go:177` covers the folder-trust and "Yes, I accept" gates and nothing else.
   `TrustDismissSequence` at `startup.go:248` accepts only those gates.
   A login prompt on an account that is not logged in reads as pending until the startup deadline, and the run then dies.
   The phrase came from the review's summary, but in the record it is the fixer's claim.
   A wording that holds: "dismissing a trust gate as the start path does".

2. **LOW.** Changed text: D7, "So a `Wait` process that dies between steps 1 and 4 leaves no run read as relaunching past the bound", read together with "reads a relaunching record within its bound as a live running run".
   The bound only takes effect when an attach or `Wait` reads the record.
   Until then the record still reads relaunching past the bound, and the new predicate does not say how other readers treat a record past its bound.
   `StopStepRuns` and the carry-over reader are examples.
   Evidence: the bound sentence names only "an attach or `Wait` that reads" as the actor, and nothing sweeps the mark on its own.

These claims hold:
- `Wait`'s probe leaves the events proof off: `wait.go:1453-1455`.
- A stale session-start line satisfies the line half: `wait.go:1449`.
- The `turnStartedAfterSessionStart` short-circuit exists: `wait.go:1498`.
- `StopStepRuns` skips a strand that is not removable: `stepstop.go:81`.
- The record has a started flag: `markStarted` at `wait.go:1566`.
- Every comparison against `runOutcomeRunning` is in `internal/shuttleengine`. A grep outside that package found none.

Tool calls:
- A grep for `"running"` and `runOutcomeRunning` outside `internal/shuttleengine`, and a check of `claudeengine` for startup files. My context lacked the readers outside the package and the startup classifier's file.
- A grep of `claudeengine/startup.go` for gate kinds, and of `runIsLive`. My context lacked which gates `Startup` recognizes and what `runIsLive` tests.
