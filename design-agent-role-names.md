# Webster's strand is named webster, and each module names its agents with its own constant

`agent-role-names` · Planned · chore

## Goal

An operator reading `lyx reed list` sees each agent named for what it is.

## Today

- `internal/agentname` forms `<shortname>[:<slug>]:<role>`, and each spawning module passes its own role, mostly as an inline literal in its spawn spec.
- Webster spawns its orchestrating session with `string(RoleMaster)` (`internal/websterengine/runlevel.go`), the same value that keys its model in `webster.yaml`, so the strand shows as `...:master`.

## Fix direction

- Webster's orchestrating strand gets the role `webster`, separate from its model key; whether the `master` model key in `webster.yaml` is renamed too is for Discussion, and a rename must keep existing configs loading.
- Each module keeps owning the roles it spawns: the role is a named constant in that module, beside its spawn site, not an inline literal.
  There is no central role list; `agentname`'s role set stays open, so a new module with its own agent touches no shared file.

## Done when

- A webster run's orchestrating strand is named `<shortname>:<slug>:webster`.
- Every spawn site takes its role from a named constant in its own module.
- The Agent Name Invariant in `CONSTRAINTS.md` states that the spawning module owns its role, and CLAUDE.md's Merriam terminology entry stays accurate.

## Depends on

`webster-recovery` (both edit `internal/websterengine`) and `run-parentage` (both edit how agent names are formed and the Agent Name Invariant).
