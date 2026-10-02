# Spawned agents' role names live in one place, and webster's strand is named webster

`agent-role-names` · Planned · chore

## Goal

An operator reading `lyx reed list` sees each agent named for what it is, and a reader finds every role a lyx module spawns in one place.

## Today

- `internal/agentname` forms `<shortname>[:<slug>]:<role>`, but each spawning module passes its own role literal (`driver`, `discussion`, `plan`, `rework`, `burler`, `friction`, `describe`, `conflict`, `judge`, `triage`, `targeting`).
- Webster spawns its orchestrating session with `string(RoleMaster)` (`internal/websterengine/runlevel.go`), the same value that keys its model in `webster.yaml`, so the strand shows as `...:master`.

## Fix direction

- Webster's orchestrating strand gets the role `webster`; whether the `master` model key in `webster.yaml` is renamed too is for Discussion, and a rename must keep existing configs loading.
- Every role a lyx module spawns is declared in one list, and a test fails when a spawn site uses a role outside it.
  Discussion decides where the list lives, given `agentname` imports the standard library only and calls its role set open.

## Done when

- A webster run's orchestrating strand is named `<shortname>:<slug>:webster`.
- Every spawn site takes its role from the one list, pinned by a test.
- The Agent Name Invariant in `CONSTRAINTS.md` names the list, and CLAUDE.md's Merriam terminology entry stays accurate.

## Depends on

`webster-recovery` (both edit `internal/websterengine`) and `run-parentage` (both edit how agent names are formed and the Agent Name Invariant).