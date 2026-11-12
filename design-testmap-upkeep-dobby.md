# The prime dobby starts an upkeep pass on remaining work, and an anchor move ends it

`testmap-upkeep-dobby` · Task · feature, testkit, gate, dobby · running · PR-Review

Decision record: [Sketch](decision-testmap-upkeep-dobby.md)

Filed from the Sketch `testmap-upkeep-design`.

Takes: testmap-upkeep-runs' lym gate testmap --upkeep, which registers itself as RoleTestMap and exits on a failed registration, and the exit record's planned HEAD and queued count; today's testMapStep, LandingsSinceTestMap, test_map_every and test_map_batch in dobbyengine, startTestMap in beamcli, dobbyPassInFlight in fabriccli
Delivers: the start rule on no exit record, a moved HEAD or queued work, held off on a hub below two slots for queued work alone, while a pass or its unregistered child lives, and while an anchor move stands; the failed-registration exit logged once per episode; test_map_every, test_map_batch and the landing count retired, the landing count still readable; test_map_upkeep (CategoryNone, true by default); dobbyengine's rewrite of test_map_every: 0 to test_map_upkeep: false, run by beam's configreg Migrate and by a new configengine.Spec per-layer migrate hook that Load applies before merging and DropRetired; dobbyPassInFlight no longer counting a RoleTestMap process
Exports: the dobby key test_map_upkeep and configengine.Spec's per-layer migrate hook

The part changes these packages:

- internal/dobbyengine (about 300 changed lines)
- internal/configengine (about 120 changed lines)
- internal/configreg (about 40 changed lines)
- internal/beamcli (about 120 changed lines)
- internal/fabriccli (about 40 changed lines)
