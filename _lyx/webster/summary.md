# Prompt-cache TTL per agent role

Batch 01 (`01-shuttle-ttl-config`) recorded `done` at head `bcf4d90e1b08b0864946e221a4c6731ce16d2a20`, in `elapsed` time not reported by the digest.

It covered the plan's three cards:

- `shuttle.yaml` and `shuttleengine.Config` carry the TTL default and the per-role open map.
- The `shuttle` registry entry in `configreg` declares the per-role map an open map.
- `claudeengine.Prepare` resolves, validates and logs the role's TTL and puts it on both the launch and resume pane lines.

## Deviations

The digest lists these files as changed outside the declared file-ops:

- `internal/shuttleengine/claudeengine/command.go`, `internal/configreg/configreg.go` and `internal/configreg/configreg_test.go`: the plan names symbols, not these files, so they are plan targets.
- `internal/shuttleengine/wait_test.go`: the new map field made a `Config{}` equality comparison uncompilable.
- `internal/testkit/envkit/envkit.go`: the new nil-legal map field needed an entry in `nilLegal`.
