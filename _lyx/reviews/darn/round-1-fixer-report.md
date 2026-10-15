# Fixer-report, round 1

## Changes

- F1 (fixed as proposed): `internal/reedengine/render/types_test.go`, in `TestDisplayMarshalsWithoutRetiredKeys`.
  Added an assertion that a `Display` with `KeepViewer` false marshals without a `keepViewer` key.
  The round-trip `want` now sets `KeepViewer: true`, which pins that the field persists and reads back.
  `lyx gate test ./internal/reedengine/render` is green.

## Disputed

None.

## Deferred

None.
