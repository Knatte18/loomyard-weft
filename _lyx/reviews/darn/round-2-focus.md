---
round: 2
focus:
  - "In internal/configengine, the filled-keys Info log drops keys under a declared per-name map (the `slices.DeleteFunc` filter near config.go:207). Settle whether a test with a Spec declaring PerNameMaps covers both cases: a base file lacking a name under the map logs no such key, and a base file lacking the per-name map path itself still logs that path."
  - "In internal/configreg/configreg.go, the PerNameMaps field comment and internal/configcli/configcli.go, the `lyx config reconcile` Long help text: settle whether each states what reconcile now does with per-name maps (keeps only names the file changed, never copies a template name in, keeps entries the template lacks, reports a template-equal entry under `dropped`, and `--apply` rewrites on an add, remove or drop)."
---

Targets the round after round 1, which found an untested log filter and two stale texts.
