```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the fix covers every member of each set a finding rests on.

**Findings left unchecked:** none. I checked the reconcile callers for F1; F2 and F3 I checked from context.

**LOW: the F1 repair does not reach every reconcile that can write the unloadable file**
- **Changed text:** `internal/configreg/configreg.go`, the `ArrasDefaultReinterpretationID` registration, and the sentence added to `internal/runprofile/doc.go` ("a one-time reinterpretation then drops a `defaults.arras` naming a profile left with no arras block").
- **Evidence:**
  - F1's premise covers "neither reconcile caller", but only `hubreconcile`'s walk applies reinterpretations.
  - `lym config reconcile --layer default --apply` (`internal/configcli/hubwide.go`) and `lym fabric reconcile` (`internal/fabriccli/fabric.go`, `runReconcile`) also follow the template through `ReconcileHubWideAt`.
  - My search for `ReconcileBeforeLoad`, `EnsureOrWarn` and `PersistentPreRun` shows neither verb running the walk first.
- **First case:** if either verb is the first to reconcile after the upgrade, it commits `defaults: {arras: light}` with no `light.arras` block. Profiles then fail to load until some later verb runs the walk. The marker is still absent at that point, so the walk repairs the file. The failure is temporary.
- **Second case:** the marker is shared across the hub, per the `reinterpret.go` doc in `hubreconcile`. An older binary's walk adds the arras blocks back (PATTERN-config-layers, "Keys the template lacks"). If the operator then sets `defaults.arras` again and the newer binary's walk drops the blocks, nothing repairs the file. This is a narrow multi-machine edge.

F3: the remaining `Settings(profile, "arras")` assertion in `TestTemplateLadder` does not duplicate `TestLoad`, so there is no other duplicate to remove. F2: the summary now names `configsync`, `hubreconcile`, both rewritten files and the handling of a kept default.

Tool calls:
1. Bash: searched for the callers that run the walk, and read `hubgeom.ReconcileBeforeLoad` and the `clone.go` walk call. My context lacked which verbs run the walk.
2. Bash: searched for `ReconcileBeforeLoad` and `PersistentPreRun` in `configcli`, `fabriccli` and `cmd`. My context lacked whether the two reconcile verbs trigger the walk.
