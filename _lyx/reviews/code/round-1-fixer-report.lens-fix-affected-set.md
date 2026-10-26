```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran the search behind F3 (stale orch-start wording) and the one behind F2 (sites that touch the await marker).

No findings.

- **F3:** a search of non-test Go in `internal/orchengine` and `internal/orchcli` for "orch start" wording finds three hits.
  - `internal/orchengine/pause.go:165` uses "the orch started under it", which is not the removed start command.
  - `internal/orchcli/start.go:576` documents `launchOrch`, which still starts the orch session.
  - `internal/orchcli/notify.go:49` says "live orch starting at from", which is a walk, not a start.
  - The three edited comments cover the sites the review named, and no stale site is left.
- **F2:** every other reader or writer of the await marker, outside the `beamawait` files, is unaffected by the new second line.
  - `beamup.go:53` removes the marker file whole, so a notice recorded earlier is cleared at the next Beam-Up.
  - The path wiring in `hubgeom`, `shedrecipe` and the test fixtures only passes the path along.
  - `markerStands` checks existence only, so the extra line does not disturb it.
- **F1:** this finding is one site, the `refuse` status-write failure branch.

Calls made:
1. `grep -rniE` for "orch start" wording over `internal/orchengine` and `internal/orchcli` (non-test): reran F3's search for members the review did not name.
2. `grep -rnE` for `AwaitMarker` and `await-entered` across the repo outside the `beamawait` files: found every other user of the marker F2 now writes a second line into, which my context lacked.
