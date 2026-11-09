# The prime parent's dobby starts lym gate testmap after every test_map_every landings

`testmap-dobby-refresh` · Task · feature, testkit, gate · running · Drift-Baseline

Decision record: [Sketch](decision-testmap-dobby-refresh.md)

Depends on: [`testmap-verb`](design-testmap-verb.md)

Filed from the Sketch `redundant-test-finder`.

Takes: testmap-verb's verb
Delivers: per the operator's decision after the Motif (2026-11-08, in the Sketch's board body), which replaces D6's trigger and supersedes the trigger clauses of the first carried criterion: a dobby tick step on the prime dobby that counts landings as the board tidy does and, after every test_map_every landings, starts `lym gate testmap --max <test_map_batch>` on the packages stale then, leaving what the cap left unbuilt for the next refresh; no prime-HEAD record and no exit-record trigger; started through a spawn dep registered as runproc.RoleTestMap under runproc.HubOwner, none while a live test-map record exists, a restarted dobby included; the child ended on stop through runproc.EndRole; dobby config keys test_map_every beside board_tidy_every, where 0 never refreshes, and test_map_batch; no test_map on/off key
Exports: runproc.RoleTestMap

The part changes these packages:

- internal/dobbyengine (about 200 changed lines)
- internal/beamcli (about 80 changed lines)
- internal/runproc (about 10 changed lines)
