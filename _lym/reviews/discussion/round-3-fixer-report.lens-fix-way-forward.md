```yaml
lens: fix-way-forward
outcome: reported
```

Task: check that the ways forward the F13 fix introduced can be carried out and lead to progress.

No findings.

Each path the fix introduces moves the run forward:
- **Crash between the row's reroute verdict and shed's persist:** `current_producer` stays at PR-Review, as `run.go:405` describes. The next `Call` finds the mark, and the newest PR-Review entry is not a reroute naming the mark's file. So the row reroutes again, and shed persists the entry this time.
- **Resume after a reroute-cap halt:** the halt entry is `Stuck` with `RerouteCapped` (`reroute.go`). The resumed row therefore keeps the mark and reroutes. `rerouteCount` stops counting at `RerouteCapped`, so this reroute is under the cap and goes through.
- **Resume after the dirty-tree Stuck:** that entry is a plain `Stuck` one. The row keeps the mark and reroutes once the tree is clean.
- **Re-entry after a persisted reroute:** the newest PR-Review entry is the reroute naming the mark's file, so the row clears the record and starts a fresh session. After the clear there is no mark, so a later re-step does not reroute. A second send-back in the same session writes a new file `pr-findings-<n+1>`, so a crash during it cannot be mistaken for the earlier send-back's reroute.
- **Reading shed status inside `Call`:** the loop holds the run lock across `Call`. But `state.ReadJSON` takes only the status lock (`StatusLockPath`), and only for the read. A status reader injected into the row can therefore read the status without blocking the loop.
- **The new criterion:** each case can be set up by hand. A missing reroute entry stands for the crash, and a `RerouteCapped` entry stands for the cap halt.

Tool calls:
- One grep of `internal/shedengine/run.go` for locks and of `internal/landingshed` for status paths. My context showed neither whether the run lock is held across `Call` nor which lock a status read takes.
