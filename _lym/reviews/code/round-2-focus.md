---
round: 2
focus:
  - "In internal/beamcli/dobbydeps.go, startTestMap starts a detached coverage-map upkeep pass: check where the pass's stdout and stderr go, and settle whether the pass can still run to completion after the dobby that started it exits or re-execs."
  - "In internal/fabriccli/anchordeps.go, dobbyPassInFlight decides whether a pass is in flight at an anchor move: check that a test pins that a registered RoleTestMap child of the dobby with no held run lock counts as no pass in flight, and an unregistered child still counts."
  - "In internal/orchcli (doc.go, stop.go, cli.go and the beamdown error text), settle whether any wording still calls the ended process a test-map refresh the dobby started rather than the coverage-map upkeep pass that registers itself."
  - "In internal/dobbyengine/state_test.go, check that a test reads a state record holding landings_since_test_map without error and keeps the other fields."
  - "In internal/configsync reconcile tests, check that a test drives the reconcile walk over a hub base file holding test_map_every 0 and asserts it ends as test_map_upkeep false."
---

The focus entries cover the sites round 1 raised; the reviewer still reads the whole diff.
