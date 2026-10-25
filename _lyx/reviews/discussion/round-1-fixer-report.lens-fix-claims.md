```yaml
lens: fix-claims
outcome: reported
```

Task: check that each claim the fix added or rewrote holds against the code at HEAD.

**1. MEDIUM.** A pause can still hold a pass in the land-lock poll, so two new claims are false.
- **Text:** D4 says the land-lock poll "ends at a pause request" and "Bound: a pass stays in flight only through those waits, never through a pause or an owner hold". D12 says "a pass is in flight at a lift only in one of D4's bounded waits".
- **Evidence:** Land's `PauseRequested` reads batten's own status flag, the one `lyx batten pause` writes (`internal/battencli/land.go:40`, `:146`, and the comment at `internal/battencli/wire.go:977`). It does not read tender's pause-record coverage.
- **Effect:** Under a covering tender pause entry, `acquireLandLock` (`internal/battenshed/land.go:352-382`) keeps polling with no time limit. A pass can stay in flight through a tender pause, and the lift pass waits on it.
- **Second contradiction:** D4 itself calls the land-lock poll the one wait with no time limit, yet D12 counts it among the "bounded" waits.

**2. LOW.** Not all of the rest of `childWait` is a fresh reading.
- **Text:** D4 says "The rest of `childWait` is each check's fresh reading and stays per pass."
- **Evidence:** `revival` (`driverRevival.startFallback`, `internal/battenshed/innerrun.go:839-843`) is set when "a revive failed in this episode". That is episode state, not a fresh reading. `reason` holds "the wait reason last written to the reason file" (`innerrun.go:566`), which is memory too.
- **Effect:** Kept per pass, the halted reason drops its `lyx loom start` fallback on the next pass after a failed revive. This is a small hole in the inventory F3 asked for.

All the other claims I checked hold:
- The three readers of `battenAlive` (`claim.go:83`, `skip.go:48`, `skip.go:105`).
- The `claimClear` arms of `settleClaim`.
- `EndBeforeSessionDown`'s caller.
- The owner trigger's three cases.
- Row notice spacing of 30 seconds with at most two sleeps (`noticeMaxAttempts = 3`).
- The ten-minute prime-lock and prime-dirty bounds.
- `LayerWorktree` exists.
- Q17's per-process awake clock (`MonotonicAwake` takes `time.Now()` as its origin per process).

Tool calls made: 1 bash call. It read the readers of `battenAlive`, the `driverRevival` type, `MonotonicAwake`, the worktree config layer and batten's `PauseRequested` wiring. My context had none of these code sites.
