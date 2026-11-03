---
round: 1
focus:
  - "internal/envconfig/config.go, internal/configengine/layers.go, internal/yamlengine/layers.go: does the machine layer accept `env` values while `tiers` stays refused as executable, and does the machine value win per variable over the hub value?"
  - "internal/gatecli/run.go: is precedence a tier's own `tiers.<n>.env`, then the env module, then the inherited environment, for tier runs and for the shared build?"
  - "internal/driftengine/config.go, internal/driftcli/wiring.go, internal/loomcli/driftwiring.go: do both Drift capture and compare processes receive the env module's variables, and what happens when the env config fails to load?"
  - "internal/reedengine/panebin.go, geometry.go, spawn.go, internal/hubgeom/hubgeom.go: do agent session panes export the variables, with correct shell quoting and invalid variable names rejected rather than injected?"
  - "Scope against the issue's design: does every process lym starts for the hub (gate tiers, shared build, Drift capture and compare, agent sessions) get the variables, and does the diff add anything the design does not license?"
  - "Repository rules: do the new `internal/envconfig` module, `configreg` registration, `pattern/PATTERN-config-layers.md` and `docs/overview.md` follow the constraints document, and do new doc comments follow its comment conventions?"
  - "Test economy: do the new tests in envconfig, driftengine, reedengine and gatecli duplicate one another, and is each added behavior covered?"
---

Round 1 seed.
The answer key is the board entry in the decision record, which has no amendments.
