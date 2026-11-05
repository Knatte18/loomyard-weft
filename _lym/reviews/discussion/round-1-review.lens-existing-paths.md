```yaml
lens: existing-paths
outcome: reported
```

**Paths traced, per decision**

- D2 (hold boundary at Finalize, dobby holds a ready beam):
  - `battencli/beamfork.go:65` fork hold check on a hand `lym batten run <member>` reads `RunHolds` — **breaks**: Beam-Await is hold-free, so in the ready→accept window nothing stops a hand run.
  - `battencli/dobbyreads.go` `runTakesNoSlot` — survives (takes a slot from Finalize on).
  - `hubgeom/dobbygeom.go` `LandingHeld` — survives.
  - `adoptcli/deps.go` `LandingHeld` — survives (D4's record is tracked).
  - `loomcli/peers.go` `RunHolds` — survives.
- D3 (`ready` refusals): `beamcli/land.go` `markReady`, `judgeLandEntry` — survives; new refusals sit beside the entry check. `lym beam land` writes a land request only for a beam flagged ready, so it cannot bypass D3 — survives.
- D4 / D8 (tracked record, Finalize reads it): `landingshed/finalize_pr.go:171` reads the scratch record, `:350` writes it for a PR Finalize opens itself — survives only if D8 keeps reading the scratch record as D4's Rationale says. Adopt carries the record across machines — survives.
- D5: no landing path other than Beam-Describe rewrites `summary.md`; weft never merged to the parent — survives.
- D7: `dobbyengine/landnotice.go` existing `notify_on_land` notice — survives; D7's bound names the double notice.
- D10 (goto keeps the ready record): `dobbyengine/land.go:46-50,79-87` (persisted land episode) and `pawlengine/beamawait.go:306` (accept removes the dobby-stop marker) — **breaks**.
- D11 (crash recovery via dobby): `pawlengine/beamawait.go:99-114,190-195`, `dobbyengine/land.go:46-50,79-87` — **breaks**.
- D12: orchcli `RunHolds` readers (`ownerpause.go:70`, `refusal.go`) — survive.
- D13 (move off retired row):
  - `beamcli/deps.go:169` + `beamcli/doc.go:52` (the only existing ready-record removal, which commits the landing directory) — **breaks**: D13 states no commit.
  - The dobby's land episode after the move (ready record gone → reading fails → `Land` cleared) — survives.
  - `beamawait.enter` re-entry after the move — survives.

**Findings**

1. **MEDIUM · design · D11** — The dobby will not rewrite the land request after the crash D11 recovers from.
   The dobby records the episode as acted and persists it (`internal/dobbyengine/land.go:79-87`, `state.go:43`); a restarted dobby skips any passing reading while `Land` is recorded and not refused (`land.go:46-50`). After a crash between the accept's removals and Done, the ready record still stands and nothing works under the beam, so the reading keeps passing and the restarted dobby never asks again: Beam-Await waits with no land request. D11 replaces today's entry notice, which names `lym beam land` (`internal/pawlengine/beamawait.go:194`), and is silent on whether that notice stays.

2. **MEDIUM · design · D10** — The bound "the dobby lands the beam again once nothing works under it" breaks after a goto from Finalize.
   The accept already removed the dobby-stop marker (`internal/pawlengine/beamawait.go:306`), so D11's entry check, which starts the dobby only when that marker stands, leaves it down; and even once the operator starts it, the persisted acted episode (`internal/dobbyengine/land.go:46-50,79-87`) keeps it from asking for the landing. The working way forward is a hand `lym beam land`, which D10 does not name.

3. **MEDIUM · design · D2** — The bound "a member filed in that window never starts" is false.
   A hand `lym batten run <member>` is gated only by `RunHolds` on the beam entry (`internal/battencli/beamfork.go:65`), and `RunHolds` reads Beam-Await as hold-free (`internal/boardengine/runlock.go:42`). So between `lym beam ready` and the accept a member starts by hand and can land into the ready beam, moving the beam head past the head D3 checked against the pull request. The D2 bound names only the dobby's start hold.

4. **MEDIUM · design · D13** — The move removes the tracked ready record and writes D4's tracked record but is silent on committing either.
   The only existing ready-record removal commits the landing directory (`internal/beamcli/deps.go:169`, `internal/beamcli/doc.go:52`), and D4 has its own writers commit; a shed step that deletes or writes these files without a commit leaves the beam's records worktree dirty.

**Tool calls**

1. Read `internal/beamcli/land.go` (header, `markReady`), `hubgeom/dobbygeom.go`, `adoptcli/deps.go`: to check whether `land` bypasses the ready refusals and how the other `RunHolds` readers judge the hold.
2. Grep `CanLand`, dobby `Land` state and `RemoveReady` callers: to find the dobby's land-episode reset and every ready-record removal.
3. Read `internal/dobbyengine/land.go:66-100` and `state.go`: to confirm the land episode persists across a dobby restart.
4. Grep `internal/landingshed/finalize_pr.go`: to check how Finalize's already-landed path reads its PR record.
