# Lenses resolve from a neutral base plus a language layer

Both batches reported `status: done`, with no departures and no warnings.

## Batch 01-configengine-overlay-load

Head `1aa8ded33ba553931f21a23fcfe1cc0dc9bb731d`.
The batch covered cards 1, 2 and 3: the spec overlay and effective-template loading in `configengine`, Set and Edit judged against the effective template, and `lens_set` wired into lenses with the `go` layer.
The scope guard listed these deviations: `internal/configengine/edit.go`, `internal/configengine/overlay.go`, `internal/configengine/set.go`, `internal/configreg/configreg.go`.

## Batch 04-lenses-language-layers

Head `ad45cfb72fbed2ca589c326293ab546bbeaf1e6d`.
The batch covered cards 4, 5 and 6: the neutral base with the `csharp` and `python` layers, `configsync` judging against the effective template, and `remove` in `lenses.Load`.
The scope guard listed these deviations: `internal/configsync/configsync.go`, `internal/configsync/configsync_test.go`.
