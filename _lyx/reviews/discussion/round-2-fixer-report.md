# Fixer report — discussion, round 2

## Fixed

- **F1 (MEDIUM, design), D5's non-integer premise.**
  Confirmed against the code: `configengine` runs `yamlengine.Resolve` before the engines unmarshal, so an `${env:...}` marker reaches the loader as a number, and a null decodes into the `int` fields of `shuttleengine.Config`, `orchengine`'s config and `loomengine.Config` as 0, which each loader raises to its floor.
  D5 now states each shape: a null is raised to the floor and reported with old value `null`; an env marker is left untouched with no line, and the loader may still raise and warn; any other non-integer such as `abc` is left untouched and the loader rejects it.
  None of them makes the hook error.
  The reconcile acceptance criterion names the null raise and the env-marker no-line case.
- **F2 (LOW, decision), bare `fabric unwire`'s reader classification.**
  Confirmed: `FabricReaderVerbs` (`internal/fabricengine/refscanner.go`) and `readerVerb` (`internal/fabriccli/roleguard.go`) admit bare `prune` and `cleanup`.
  D6 now decides that bare `unwire` stays out of the reader set and gives the reason: no sandboxed role needs the probe, and joining would widen the guard and `IsReadOnlyCommand`'s allow-list.
- **F3 (LOW, consistency), R4's second-window claim.**
  Confirmed: `openTerminalOnce` in `internal/battenshed/innerrun.go` runs only after a successful spawn.
  R4 now says the reseed clears the marker either way, and a second window opens only when InnerRun then spawns the child.

`support-log.md` gains the rejected alternatives behind the F1 and F2 choices and ledger entries Q10 and Q11.

## Disputed

None.

## Deferred

None.
