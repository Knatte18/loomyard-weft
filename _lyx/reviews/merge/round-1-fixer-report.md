# Fixer report — merge round 1

## Changes

- F1 fixed as proposed.
  `scalarOrListNode` in `internal/yamlengine/set.go` now accepts flow-list entries that are scalars or mappings of scalars (new `isScalarOrScalarMapping`); the text-after-bracket check stays, and the refusal says "scalars or mappings of scalars".
  The rule is restated in `set.go`, `doc.go` and `layers.go` of `internal/yamlengine` and in `internal/configcli/doc.go`.
- Tests: a `set_test.go` row sets an open-map path whose template value is `[]` to a flow list holding one `{name, model}` mapping and expects the flow list back; a refusal row pins a mapping that holds a list; `TestSetDiscussionAdvisors_LoadsBack` in `internal/loomengine/config_test.go` sets `discussion_advisors` through `configengine.Set` and loads it through `LoadConfig`.
- Commit: `yamlengine: fix F1`.
  `lyx gate test` over `yamlengine`, `loomengine`, `configcli` and `configengine` is green.

## Disputed

None.

## Deferred

None.
