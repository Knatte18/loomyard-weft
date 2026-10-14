# Fixer-report — merge round 1

## Changes

- F1 (NIT), fixed as proposed, commit `75dcf69d3`:
  the null-tag sentence in the `SetValues` doc comment (`internal/yamlengine/set.go`) and in the `SetSparse` doc comment (`internal/yamlengine/layers.go`) now starts "A scalar leaf outside a scalar-or-list open-map path", so neither reads as contradicting the scalar-or-list rule.
  Comment-only change; `go build ./internal/yamlengine/` passes.

## Disputed

None.

## Deferred

None.
