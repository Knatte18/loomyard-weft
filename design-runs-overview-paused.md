# Runs overview shows a paused run as paused

`runs-overview-paused` · Note · enhancement, reed, batten

## Seen (2026-10-12, from `lyx-halt`'s parent review)

`lyx-halt` adds a pause, hub-wide or per run, recorded with a `paused_since` time.
The runs overview pane from `runs-overview` reads run states without it, so a paused run shows as running.
Both tasks were in flight when this was found, and `runs-overview`'s entry was claimed, so this note carries the gap.

## Want

The runs overview shows a paused run as paused, with the time since it was paused, reading `lyx-halt`'s record.
Whichever of the two tasks lands second does this, or it becomes a small task after both land.
