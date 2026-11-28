```yaml
lens: fix-affected-set
outcome: reported
```

Task: affected-set check of the fix change set.

**Unchecked list:** none. F1 to F4 and F8 were rerun as searches. F5 to F7 are single-site coverage gaps, not sets.

**MEDIUM: a live dobby can hold a wait that nothing retries or clears**
- **Changed text:** `hubBeamCatchUpWaits` and its comment (`internal/beamcli/dobbydeps.go`), the `addCatchUp` loop in `internal/dobbyengine/pushsummary.go`, and the Design decisions bullet "A stopped dobby's record is left out, since nothing retries or clears it" in `_lym/landing/summary.md`.
- **Problem:** the fix treats "live" as meaning "the record is current", but a dobby that re-execs while a catch-up is owed is live and never retries it.
- **Evidence:**
  - `catchUpOwed` is held only in memory.
  - `recordStart` (`internal/dobbyengine/loop.go:493-514`) returns early on a re-exec without calling `oweCatchUp`.
  - `State.CatchUp` is cleared only in `catchUpSucceeded`.
- **Consequence:** after a binary change during an outage, the record keeps `Reason: origin` until a sleep, a hand-over or a pass exit happens to owe a new catch-up. Until then, the prime's summary reports the wait with ever-growing minutes and its period never reaches the all-clear. The prime's own record already behaved this way before this fix; the fix extends it to every beam dobby.
- **Possible remedy:** in `recordStart`, owe the catch-up when the record it read has a `CatchUp` wait, on a re-exec too.
- **Smaller case of the same kind:** a beam dobby under an adopt hold is live and does not run its catch-up (`loop.go:727`), so its old origin wait is also still listed.

No findings on the other sets:
- **Exit-code consumers:** `passExitCode`, `passExited`, `firstPassExited`, and the doc comments in `doc.go` and `geometry.go` all handle the new code.
- **Notice-content text:** the sites found agree with the change.
- **Origin-wait retry:** it stays bounded in the paths traced, including a re-exec losing `originStops`.

Tool calls (3):
1. `git grep` for pass exit codes and their consumers: the context held only an `internal/`-scoped search for some of the names.
2. `git grep` for push summary and catch-up wait readers outside the changed files: the context had no repo-wide search for them.
3. `git grep` for catch-up owing and refused-start sites: the context had not enumerated them after the change.
