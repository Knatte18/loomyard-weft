# Launch manifests per engine, a parity test per seat, and lym shuttle manifest

All three batches recorded `done`; no recovery, rename or pattern departure occurred.

## Batch 01, heddle-standing-keys (head 0fe1c803)

Standing passages and heddle file entries carry their slug and the engine they are keyed to.
The manifest record type, its comparison, the copy a launch keeps and the scans that skip it were added, and Claude's `Prepare` writes its manifest.
The scope guard listed these deviations: `internal/heddleengine/render.go`, `render_test.go`, `target.go`,
`internal/shuttleengine/manifest.go`, and `internal/shuttleengine/claudeengine/manifest.go`, `manifest_test.go`, `pluginhook.go`.
Departures: none.

## Batch 04, codex-manifest (head affd81e7)

Codex's `Prepare` writes its manifest, the engine set can be built with a given hook runner and sandbox probe,
and the new `internal/shuttleparity` package holds the seat catalog and the per-seat parity test.
Deviations: `internal/shuttleengine/codexengine/manifest.go` and `manifest_test.go`, `internal/shuttleengines/shuttleengines.go`,
and `internal/shuttleparity/` `compare_test.go`, `prepare.go`, `seats.go`, `seats_test.go`.
Departures: none.

## Batch 07, producer-table (head 35412c92)

The producer table, the scan for unlisted launch-spec literals, seat tables and seats, the `lym shuttle manifest` verb,
and the `PATTERN-engine-parity` entry with its background were added.
Deviations: `cmd/lym/treesitterlink_integration_test.go`, `internal/shuttlecli/manifest.go`, `internal/shuttleparity/producers_test.go`.
Departures: none.

## Scope guard

Every deviation listed above is a file the batches' own cards touched in support of their targets; the digests carried no warnings.
