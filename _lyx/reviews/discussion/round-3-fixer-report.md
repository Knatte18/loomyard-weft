# Round 3 fixer report

## Fixed

- F1: D10 now says the config error is raised by every `lyx orch` verb (the group's `PersistentPreRunE` loads the config), `status` included, and the acceptance criterion matches.
- F2: D16 now says `State.Stuck` also carries an idle-phase hold reason, set and cleared only by the idle probe's result.

## Deferred

None.
