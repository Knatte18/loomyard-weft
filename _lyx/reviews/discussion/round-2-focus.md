---
round: 2
focus:
  - "Config shape in `decision-record.md` D1: check against `internal/darnengine/config.go` (`ConfigOpenMaps`, `ConfigSpec`) and `internal/yamlengine/doc.go` whether `tree_scope` can be set by `lyx config darn --set tree_scope.<tree>=<target>` and survives `Reconcile` after a binary change, and whether the record names how scope is turned on and off given that `setChecked` in `internal/configcli/configcli.go` and `setHubWide` in `internal/configcli/hubwide.go` roll back a write after which `LoadConfig` fails with a new error."
  - "Publish skip seam in D4: check whether the record says when the seam classifies the task diff at Publish, given that `internal/landingshed/publish.go` reads `PublishVerify` from config deps wired once per process in `internal/loomcli/darnwiring.go`, and what happens when a Darn-Write rework after a PR-Gate rejection adds a production path to a diff that was tree-scoped at the first Publish."
  - "Categories and tree-key rules in D1: check that the record's claim about the config categories of `verify`, `tree_scope_verify` and `tree_scope` matches `ConfigSpec` and `internal/configengine/layers.go`, and that its tree-key refusal list covers keys `git diff --name-only` output (`internal/fabricengine/configchanges.go`) can never match and keys that collide after the trailing `/` is trimmed."
  - "D5 and D4 rationale: check whether `lyx gate test --guards` states what it does on a `FindGuardTests` error (`internal/impactset/guards.go`) and which directories it scans, and whether D4's claim about how often the whole-module suite runs agrees with `impactset` falling back for any changed non-`.md` file outside a Go package directory (`internal/impactset/doc.go`)."
---

Each entry names a place to look and the question to settle against the code it cites.
