# Card 8 — callers-address-roles

**Edit:**
- `internal/loomengine/driver.go`
- `internal/loomcli/bootstrap.go`
- `internal/loomcli/driverspec.go`
- `internal/orchcli/start.go`
- `internal/battencli/wire.go`
- `internal/loomcli/bootstrap_test.go`
- `internal/loomcli/driverprompt_test.go`
- `internal/loomcli/start_driver_test.go`
- `internal/orchcli/cli_test.go`
- `internal/battencli/wire_test.go`

**Uses:**
- `internal/agentname/agentname.go`
- `internal/loomcli/driverprompt.go`

**Intent:**
Every caller that names a strand by a fixed literal now passes a role segment, and every caller that finds a strand by name matches through `agentname.Matches`, since `StrandStatus.Name` is now a full name.

- `loomengine.LoomDriverStrandName` becomes `agentname.RoleDriver` (`driver`), the role the hub addresses as `<slug>:driver`; its doc keeps the reason the value must never vary.
  `driverTeardownCommand` (`driverprompt.go`) is built from it and so now removes `--name driver`, which reed reads relative to the worktree.
- `loomengine` keeps the old literal as `LegacyLoomDriverStrandName = "loom-driver"`.
  Its doc says it names a driver strand recorded before this task, that no add ever uses it, and that it exists because `agentname.Matches` reaches a legacy name only by equality, so a `driver` query alone never sees it.
  Both driver lookups, loomcli's `resolveDriverStrandAction` and batten's `driverAliveFrom`, match a strand named either `agentname.RoleDriver` (through `agentname.Matches`) or exactly the legacy literal.
  A live pre-task driver therefore still counts as live, so a re-entered `lyx loom start` spawns no second driver beside it, and batten's inner-run wait still grants it the exit grace.
  A dead one is still found and removed by GUID before a relaunch, so a later reed resume never revives it.
- `orchcli`'s `orchStrandName` becomes `agentname.RoleOrch`, so the hub's strand is `<code>:orch`; `orchStrands` matches with `agentname.Matches`.
- `loomcli`'s `statusStrandDisplayName` stays `loom-status`, now a role segment; `findStatusStrand` and `removeStatusStrands` match with `agentname.Matches`.
- `battencli`'s wait on the child's driver strand (`wire.go`) matches with `agentname.Matches`.
- `driverSpec`'s doc says `NameOverride` is an explicit role, which reed refuses rather than renumbers when held, and why the bootstrap's own dead-strand removal keeps that from firing.

Tests: the bootstrap, orch and batten lookups find a strand whose name is a full name (`ly:task:driver`, `ly:orch`, `ly:task:loom-status`), and still find a legacy exact name;
`resolveDriverStrandAction` reports a live `loom-driver` strand as `driverStrandLive` and a dead one as `driverStrandDead` with its GUID, and `driverAliveFrom` reports a live `loom-driver` as alive;
the driver prompt and start tests assert `--name driver`.

**ImpactSummary:** Four lookups in three packages (loomcli, orchcli, battencli) move to `agentname.Matches` and still find every full-name strand; pre-task `loom-status` and `orch` strands stay reachable by equality, and a pre-task `loom-driver` is reachable only through the legacy constant, without which a run in flight across the deploy would spawn a second driver.

**Commit:** 8: loom, orch, batten: address strands by role segment
