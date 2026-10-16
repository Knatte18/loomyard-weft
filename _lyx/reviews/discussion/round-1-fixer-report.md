# Fixer report — discussion round 1

Every finding's premise was checked against the code: `internal/loomcli/drive.go` (tamper check), `internal/configreg/runsettings.go:60` (`module.Migrate` on run_settings), `internal/loomengine/config.go` (`MigrateConfig` keeps no record; zero `tackler_timeout_min` takes 10), `internal/loomcli/drivewiring.go:47` (`ResolveTackler` on loom's config), `internal/loomcli/tackler.go:371` (goto as current or routed-from row), `internal/loomengine/template.yaml` (four defaults spelled out), and batten seeds under `loomyard-weft/_lyx/shed/` recording `"driver": "go"`.
All held.

## Changes

All in `_lyx/discussion/decision-record.md`.

- F1 (fixed as proposed, second branch): D4's tamper sentence now states what the missing snapshot lets through (prime writes whose verdict is acted on, a resume re-entering Land or Worktree-Teardown over them) and that the only bound is the stencil's write rule plus the agent settings' denies, with the stop-key check bounding run movement only; the D4 Bound line carries it; acceptance criterion added for the write rule in the rendered batten prompt and the batten spawn's denies.
  The narrowed batten snapshot was not chosen: concurrent landings of other slugs and the orch write the prime's tracked paths, so it would hand back falsely, the reason D4 already gives for dropping the snapshot.
- F2 (fixed as proposed, option a): D5 names that `Migrate` runs on every reconcile and on run_settings, and replaces the value-equality rewrite with a one-time rewrite of the hub `loom.yaml`, keyed on all four old defaults with their old template comments, which the rewrite replaces; run_settings documents are never rewritten; a Bound line states what is reset.
  D5 now states that the loom template's four keys, `darn.yaml`'s four and `batten.yaml`'s three ship with the inherit value.
  The Migrate criterion covers the once-only rewrite, a key set apart, and a run_settings case; a criterion for freshly seeded files was added.
- F3 (fixed as proposed): D5 states that the arming module passes the run's own recipe's override for every tackler key, chosen by the seed's recipe at `lyx loom drive`'s wiring; darn criterion added.
- F4 (fixed as proposed): D2 gains the resume goto targets seam; D4's resume sentence points at it; D7 says the stencil's goto rule reads from it; the batten criterion refuses a routed-from goto.
- F5 (fixed as proposed): D2's Bound line rewritten with the reviewer's text.
- F6 (fixed as proposed): D3's premise rewritten; D4 gains the sentence on a batten run started on the old build.
- F7 (fixed as proposed): criteria added for batten's never-firing hand-over and verify-goto rows, and the rendered prompt's no-arbiter, stop-kind list and reading order.
- F8 (fixed as proposed): D5 picks that a set `tackler_timeout_min: 0` inherits the next layer in a module and takes 10 minutes in `shed.yaml`; the plan's representation question narrows to `tackler_max_spawns`; overrides criterion extended.
- F9 (fixed as proposed): D4's keep-behaviour sentence names `abandonedSession` once, under `lyx batten run`'s envelope.

## Disputed

None.

## Deferred

None.
