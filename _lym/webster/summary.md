# Config keys nest under the area they govern, named with dots

This session resumed a run whose batches were all recorded, so it began, forked and recorded nothing.
`lym webster status` reports every batch `done` with no recoveries:
01 `config-key-name`, 04 `nest-loom-tapestry`, 05 `moved-key-tables`, 09 `reconcile-before-load`,
10 `loom-key-messages`, 11 `module-key-messages`, 12 `template-comments` and 13 `contracts-key-names`.

The plan's intent for these batches is that `loom.yaml` and `tapestry.yaml` hold one map per area,
move tables carry hubs' old values to the new paths and refuse old names,
and messages, comments, stencils, specs, recipes and docs name config keys in full.
That is the plan's statement, not a report of the diff.

## Deviations, departures and scope guard

This session read no batch digest, so it has no deviations, departures, renames or scope guard result to report.
The earlier session's summary is archived beside this file as `summary-20261126T131242Z.md`.

## Parent decision still open for review

The parent decision "Name in full the bare keys card 11 left" has no batch of its own in this session.
It assigns the check of each listed site to Code-Review, and the run is to name every one in full before it lands.
