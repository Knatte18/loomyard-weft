---
format: 5
approved: false
language: go
---

# Plan: lenses module

Every lens and fan moves out of burler into a config module of its own, `lenses`, with its file `lenses.yaml`, which burler's reviewers and fixers, loom's twill writers and fan keys and darn's fan keys all read.
Reconcile keeps `lenses.yaml` complete per name and never prunes a lens or fan, the template's `twill-discussion` becomes the seven-lens trial fan, and a hub's `burler.yaml` migrates into `lenses.yaml` through two declared key moves at the first start verb after deploy.

## Card Index

1 — yamlengine-per-name-maps — add the per-name map kind to Reconcile, Prune, SetValues and SetSparse
2 — configengine-per-name-spec — Spec.PerNameMaps, honoured by the unknown-keys warning and Set
3 — lenses-module — create and register internal/lenses with the template, load, fan resolution and fan key grammar
4 — configsync-per-name-reconcile — reconcile adds per name and keeps and reports what the template lacks, at every layer
5 — burler-reads-lenses — burlerengine, burlercli and loomcli consume internal/lenses
6 — loom-reads-lenses — loom's fan keys and twill lenses resolve through lenses.yaml
7 — darn-reads-lenses — darn's fan keys resolve through lenses.yaml
8 — retire-burler-module — delete burler's config, deregister it and migrate burler.yaml through two key moves

## Shared Decisions

- A per-name map is declared once on the module's `configengine.Spec` (`PerNameMaps`) and mirrored on `configreg.Module` (`PerNameMaps`) beside `OpenMaps`, so configsync reads it from the module as it reads open maps, and the registry fingerprint covers it.
- A name under a per-name map that the template lacks is reported in its own list, `kept` from yamlengine and `Result.KeptExtra` from configsync, never in `removed`.
- `internal/lenses` stands beside burler's old config until card 8, so each card builds on its own; card 8 deletes burler's copy only once every reader has switched.
- D4 says a dry run reports the two moves "but not the additions".
  The moves report no additions, but a dry run over an absent board `lenses.yaml` under seeding has the module loop report the template seed's names as added, as `ReconcileHubWideAt` does for every absent repo default it would seed (`internal/configsync#ReconcileHubWideAt`, the absent-file branch).
  Suppressing that report for one module would add a special case the record does not ask for, so card 8 pins what the dry run reports instead.

## PATTERN changes

- `PATTERN-leaf-packages` — changes: Importing into a leaf package: only its closed import set, packages and sets named in the background.
- `PATTERN-config-strictness` — changes: Loading config: a caller adopts exactly one of strict `Load` or `LoadOrTemplate`, bar `runprofile`.

## verify:

go build ./...
go vet -tags llm ./internal/burlerengine/...
go test -tags integration ./internal/yamlengine/... ./internal/configengine/... ./internal/lenses/... ./internal/configreg/... ./internal/configcli/... ./internal/configsync/... ./internal/burlerengine/... ./internal/burlercli/... ./internal/loomcli/... ./internal/loomengine/... ./internal/darnengine/... ./internal/hubreconcile/... ./cmd/lyx/...
