# Round 2 fixer-report — discussion

## Changes

All in `_lyx/discussion/decision-record.md`.

- F1, fixed as proposed: D8's creel bullet now names the interactive strands (plain `--cmd claude` and `--cmd bash`) as the ones that start fresh shells, and D8's Rationale says the creel suite's interactive strands are all outside the guarantee.
  Premise confirmed: `tools/sandbox/SANDBOX-CREEL-SUITE.md` fixtures include a fake idle pane script, which runs in a non-interactive shell that reads no rc.
- F2, fixed as proposed: the Goal's #679 bullet and D3's Rationale now promise a released lock at `down`'s return and a process that ends after D3a's one re-check.
  D2's Rationale carried the same overclaim ("`down` then returns with no reed process left"); it now says `down` returns with the watchdog out of its loop.
- F3, fixed as proposed: D8's lead sentence now allows the creel suite's closing pre-condition sentence to count only the checks that remain.
  Premise confirmed at `SANDBOX-CREEL-SUITE.md` line 22: "which is why all three are pre-condition checks rather than assertions."

## Disputed

None.

## Deferred

None.
