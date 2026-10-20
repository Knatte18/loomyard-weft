# reed warns on every strand launch on tmux 3.4 about allow-set-title

`allow-set-title-warn-noise` · Note · enhancement, reed

## Seen (2026-10-20)

- On the work laptop (Ubuntu 24.04, tmux 3.4), `lyx orch start --continue` printed one `WARN reed: could not turn off allow-set-title, the pane title may drift … invalid option: allow-set-title` per strand pane it launched.
- `internal/toolfloor/doc.go` keeps `allow-set-title` (tmux 3.5) optional on purpose: `internal/reedengine/spawn.go` Warns and continues, and `lyx reed list` reports title drift.
- The fallback is correct; the Warn is noise, repeated for every strand on every launch on a supported tmux.

## Idea

Detect the tmux version once (the floor check already knows it) and skip the `set-option` below 3.5, logging the fallback once per process at debug or info, instead of a Warn per pane.
