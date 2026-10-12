# Selvage in Alt+Up/Down and Alt+z zoom

`selvage-nav` · Note · enhancement, reed

## Idea (operator, 2026-10-12)

In every reed session, Selvage is reachable like a strand: a free terminal tied to reed is useful, so the operator wants to step to it and zoom it in.

## Seen (2026-10-12)

- `internal/reedengine/doc.go`: Alt+Up and Alt+Down step through the strand panes of the strands' window and pass over Selvage; Alt+z does nothing in Selvage.
- So today Selvage is reached only by mouse, and never full screen.

## Want

- Alt+Up and Alt+Down include Selvage in their cycle, keeping the zoom, so stepping onto it while zoomed shows it full screen.
- Alt+z zooms and unzooms Selvage like a strand pane.

## Open

- Alt+Up and Alt+Down step between panes; whether they should ever step between windows is left for later (operator).
