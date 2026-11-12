---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/beamcli/dobbydeps.go:626
    summary: the detached upkeep pass writes to a pipe only the dobby reads, so a dead or re-exec'd dobby can kill the pass with SIGPIPE
    fix: In startTestMap, send the pass's output somewhere that outlives the dobby (a file under the hub's temp dir named via lymtemp, or discarded as before) instead of a tailBuffer pipe; if the error text must stay, have the pass itself write its registration-failure text to a file the dobby reads on exit. Add a test that a started pass still runs after the dobby closes its read end.
    origin: lens:code-correctness
  - id: F2
    severity: MEDIUM
    class: scope
    location: internal/fabriccli/anchordeps.go:482
    summary: no test pins that dobbyPassInFlight spares a registered RoleTestMap process, the D11 acceptance criterion
    fix: Add a row to the in-flight coverage in internal/fabriccli (anchor_test.go or a dobbyPassInFlight test) with a registered RoleTestMap child of the dobby and no held run lock, asserting no pass in flight, and a row with an unregistered child asserting it still counts.
    origin: lens:code-tests
  - id: F3
    severity: LOW
    class: consistency
    location: internal/orchcli/doc.go:48
    summary: orchcli comments and error text still call the ended process the test-map refresh the dobby started
    fix: Reword internal/orchcli/doc.go lines 48-50, stop.go lines 37, 50 and 74, and cli.go line 125 to the coverage-map upkeep pass, which registers itself; change the asserted substring in internal/orchcli/beamdown_test.go with the error text.
    origin: lens:code-affected-set
  - id: F4
    severity: LOW
    class: scope
    location: internal/dobbyengine/state.go:36
    summary: no test reads a dobby state record holding landings_since_test_map, which the record requires to stay readable
    fix: Add a ReadState row in internal/dobbyengine/state_test.go over a record holding landings_since_test_map, asserting it reads without error and the other fields survive.
    origin: lens:code-tests
  - id: F5
    severity: LOW
    class: scope
    location: internal/configreg/runsettings_test.go:65
    summary: no test drives the reconcile walk over a hub base file holding test_map_every 0, so the Migrate-before-prune order is pinned nowhere
    fix: Add a case to the existing hubreconcile or configsync reconcile test table asserting a hub base file or prime layer holding test_map_every 0 ends as test_map_upkeep false.
    origin: lens:code-tests
---

### [MEDIUM] Detached pass output goes to a pipe the dobby alone reads

**Location:** internal/beamcli/dobbydeps.go:626
**Issue:** `startTestMap` sets `cmd.Stdout` and `cmd.Stderr` to a `tailBuffer`, so the pass writes to a pipe whose read end the dobby holds. The pass is detached and meant to outlive a dead dobby until HEAD moves. Once the dobby exits, the pass's next write to fd 1 or 2 raises SIGPIPE, which ends a Go process, and no exit record is written. This follows from the code and was not run.

### [MEDIUM] D11's anchor-move behaviour has no test

**Location:** internal/fabriccli/anchordeps.go:482
**Issue:** The change sparing `RoleTestMap` is the acceptance criterion "an anchor move that finds a pass running ends it and does not wait". The diff changes no `internal/fabriccli` test, and the existing anchor tests fake the in-flight check, so reverting the line breaks nothing.

### [LOW] Stale refresh wording in orchcli

**Location:** internal/orchcli/doc.go:48
**Issue:** The doc, `stop.go`, `cli.go` and the error text still describe a refresh the dobby started and registered. The pass now registers itself. Behaviour is right; the wording disagrees with the updated beamcli doc and overview.

### [LOW] Retired landing count readability is unpinned

**Location:** internal/dobbyengine/state.go:36
**Issue:** The record says an old state holding `landings_since_test_map` stays readable. Go's decoder tolerates it, but no test guards it.

### [LOW] Reconcile-walk migration is unpinned

**Location:** internal/configreg/runsettings_test.go:65
**Issue:** The acceptance criterion about the reconcile walk is covered only through `Module.Migrate` and the in-memory load. No test runs the walk, so the Migrate-before-prune order is not checked.

## Rejected

- Lens code-tests finding 4 (the gatecli `--upkeep` registration-failure exit): its code and tests belong to the earlier part and are outside this diff.
- Non-mapping root now fails the load in `MigrateConfig` (code-correctness note): no evidence it changes behaviour, and the lens rated it low risk.
