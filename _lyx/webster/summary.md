# reed-view: two-line reed bar, segment colors, switch verb, zoom bracket

All four batches (cards 1–15) recorded `done`.
Final head: `b033d840a`.

## Batch 01 — segmentcolor-package (cards 1–5, head `b93f2bf35`)

Added the `internal/segmentcolor` leaf package (segments, palette, tmux colors) and reed's per-segment color config, refused at boot outside the palette.
Strands now get `@strand`, `@strand_color` and `@lyx_strands`, and carry their segment.
Added `lyx reed switch --next|--prev` and the two-line status formats with pane-border titles.
Deviations: `internal/reedengine/replace.go` (`ReplaceStrand` returns the resolved `Color`) and `internal/reedengine/lock_test.go` (test engine carries two segment colors).
The fork reported tmux-tier tests failing until the title-row geometry of card 8.

## Batch 06 — retire-status-line-pipeline (cards 6–8, head `f950a4cd8`)

Removed the status-line template, verb, config key and `tokenvocab`.
Added the Alt+z/Up/Down/Left/Right and status-bar click bindings, pinned at boot and attach, and adjusted the layout pins for the title row, run at every apply and in the attach chain.
Deviations: `internal/reedcli/smoke_switchseed_test.go` (renamed from `smoke_statuslineseed_test.go`) and `internal/reedengine/strandops_integration_test.go` (corrected a card-5 expectation about the replaced pane's row).
The fork reported `TestSmokeDotFill/CrossClientControl` (`tmux` tier, `internal/reedcli`) still failing: its 8-row pane no longer shows the first strand's marker under the two-line bar.
The plan's verify never runs the `tmux` tier.

## Batch 09 — zoom-bracket (cards 9–12, head `7fdef36fc`)

A zoomed strand now survives every reed step, the resize hook and the attach chain.
A revived server's sessions come back in spawn order.
Shuttle carries the segment to reed and types the provider's color command, and the driver, discussion, plan, rework, describe, conflict and friction specs name their segment.
Deviations: `internal/reedengine/lock.go`, `internal/hubgeom/hubgeom_test.go`, `internal/shuttleengine/startup_test.go`, `internal/testkit/shuttlefake/shuttlefake_test.go`, and the `seam_enforcement_test.go` import allowlists of `frictionengine`, `mergeresolve` and `landingshed`.
In the watchdog's poll mode a zoomed strand is unzoomed and re-zoomed on each tick, as the card specifies.

## Batch 13 — webster-review-segments (cards 13–15, head `b033d840a`)

Webster, burler, bouncer and treadle specs name their segment.
The orch watcher types the orch's color at start and first in every reload.
`lyx ide spawn` seeds a marked Alt-key block in VS Code's `keybindings.json`; a skipped seed never fails the spawn.
Deviations: `internal/treadleengine/seam_enforcement_test.go` (import allowlist) and `internal/orchengine/watcher_compact_test.go` (one extra tick for the color step).

## Open items

- `docs/benchmarks/test-redundancy.md` still names deleted tests; it is a generated report that no fork regenerated.
- The plan-level verify gate runs after this turn ends.
