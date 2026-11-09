```yaml
lens: code-affected-set
outcome: reported
```

**Task:** code-affected-set lens. For each change, I list what depends on it outside its own lines and whether the diff covers it.

**Set per change**
- **`runproc.RoleTestMap`:**
  - Other users of roles are `fabriccli/anchordeps.go` (`DobbyPID`, `EndDobbies` via `EndSparing`, `dobbyPassInFlight`) and `pairteardown`, `loomcli/pawlwiring.go` and `slackengine/start.go` (per-slug or relay-only). `runproc/doc.go` says "one of the `Role` constants", so it needs no edit.
  - The anchor-move paths are not covered; see finding 1.
- **Config keys `test_map_every` and `test_map_batch`:** A repo-wide grep for the sibling key `board_tidy_timeout_min` finds hits only in `dobbyengine` (`config.go`, `template.yaml`, `doc.go`), which the diff updates. No other doc, fixture or reconcile file lists beam.yaml keys. Covered.
- **`State.LandingsSinceTestMap`:** Only `tidy.go` and `testmap.go` read or write it. The `omitempty` JSON tag keeps old records valid. Covered.
- **`countLandings` and `tick`:** `loop.go` and the `doc.go` text on the anchor-move hold are updated. The harness calls `countLandings` directly. Covered.
- **`endDobby` in `orchcli/cli.go`:** The only caller is `beamDown`. The other `EndRole(RoleDobby)` callers (`pairteardown`, `loomcli/pawlwiring.go`) act on slug owners, never `HubOwner`. Prime stop is covered. Anchor move is not; see finding 1.
- **`docs/overview.md`, the `beamcli`, `orchcli` and `dobbyengine` `doc.go` files:** updated.
- **`fabriccli` `doc.go`, `anchor.go` and `anchordeps.go`:** not updated; see finding 1.
- **PATTERN entries:**
  - `PATTERN-child-processes`: the spawn is logged, waited on, registered with `ChildRecord` and ended via `EndRole`. The exception is finding 2.
  - `PATTERN-dobby-runs`, `PATTERN-test-speed` and `PATTERN-config-layers`: no violation seen in the affected set.
- **Declared PATTERN changes and `Pattern-Departure` trailers:** the constraints section declares no PATTERN change and no commit carries a trailer, so nothing is owed.

**Findings**

1. **MEDIUM, class `design`:** The prime's anchor move now interacts with the refresh, and no code or doc outside `dobbyengine`, `beamcli` and `orchcli` says how.
   - Location: `internal/fabriccli/anchordeps.go:255-262` and `:489-493`, `internal/fabriccli/anchor.go:468`.
   - `EndDobbies` runs `EndSparing(HubOwner, RolePush, RoleSync)` for the prime. The `test-map` record is neither spared nor a `RoleDobby` record, so a prime anchor move ends a running refresh. `LandingsSinceTestMap` was already reset to 0 when that refresh started, so the killed refresh is not retried until another `test_map_every` landings.
   - `dobbyPassInFlight` also counts the refresh as a pass in flight: it is a direct child of the dobby (`PPID` is the dobby pid) and not a push or sync. The anchor move therefore waits for a build-and-test run that can be long.
   - The change description states this wait behaviour, but the `fabriccli` doc and the `dobbyengine` doc bound do not mention it.
   - The summary's claim that the refresh reads as a pass in flight holds. The unrecorded consequence is that the move ends it, which loses the refresh.
   - Evidence: `anchordeps.go:489-493` shows the in-flight read. `anchor.go:468` and `anchordeps.go:258` show `EndSparing` without `RoleTestMap` in the spare list.

2. **LOW, class `design`:** `startTestMap` ends a child through `runproc.EndProcess` when registration fails.
   - Location: `internal/beamcli/dobbydeps.go:585-588`.
   - `PATTERN-child-processes` says `EndProcess` serves only the orphans `lym tender reap` ends. The pattern names `End`, `EndSparing` and `EndRole` as the only ways to end a run's processes, and the pattern's own "Enforcement" section says review covers this. An unregistered child has no record for `EndRole` to read, so the diff chose `EndProcess`, but no `Pattern-Departure` trailer records the reason.
   - Evidence: `pattern/PATTERN-child-processes.md` lines 34-49; `dobbydeps.go:585`.

**Tool calls**
1. Grep for `board_tidy_timeout_min`, `RoleSlackRelay` and `RoleDobby` across the repo: to find the other config and role users the manifest lacked.
2. Read `fabriccli/anchordeps.go` at lines 175-205, 250-275 and 455-480, plus `runproc/doc.go`: the anchor-move users of the dobby role.
3. Read `fabriccli/anchordeps.go` lines 480-520, plus a grep of `EndDobbies` and spare in `fabricengine` and `fabriccli`: the pass-in-flight scan and how the spare list is passed.
