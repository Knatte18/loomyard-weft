# Creel gets its own sandbox suite

Batch `01-creel-suite-file` covered all three plan cards and recorded `done` at head `64dee2c5019872f1781f1af93eea97d6bbcabc8d`.
The digest lists no deviations and no warnings, so the scope guard found no files changed outside the batch's targets.

- Card 1: wrote `tools/sandbox/SANDBOX-CREEL-SUITE.md` with S9 and C1-C13, and cut the shuttle suite to S1-S8.
  The fork filed board note `creel-c13-stacked-rewrite` for the C13 stacked half.
- Card 2: embedded the creel suite as `creelSuite`, dispatched it from a `creel-suite` subcommand, and added table-test rows.
  The fork reports rewriting two header comments, in `main.go` and `suite.go`, to satisfy `lint-comments`.
- Card 3: added the `creel-suite` launchers `sandbox/posix/creel-suite.sh` and `sandbox/win/creel-suite.cmd`, and named the suite in `docs/sandbox-howto.md`.

The fork reports the integration gate for `./tools/sandbox` as green.
The plan-level verify runs after this turn ends.
