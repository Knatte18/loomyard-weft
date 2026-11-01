## Goal

```markdown
## Seen (operator, 2026-10-28)

The operator runs KDE Plasma with virtual desktops and tiling.
The Konsole windows lym opens on a run's worktree (`lym reed window`, batten's terminal for each run) and the VS Code windows it opens (`lym ide`) land wherever KWin puts them, untiled and on the current desktop, so the operator moves and tiles each by hand.

## Design (operator, 2026-10-31)

Lym places no window itself: the operator's own tiling script does it.
Neve's `~/.local/bin/arrange-to-tiles.sh` (Meta+Alt+A, run through xremap) fills the active desktop's existing Custom Tiling grid with the windows on it, without changing the grid's shape.
- A machine-local config key names a command lym runs after it opens a window, such as `~/.local/bin/arrange-to-tiles.sh`: after a run's terminal window, `lym reed window` and `lym ide`.
- The window lands on the active desktop, as KWin puts it today, and the command snaps it into the grid.
- Lym knows nothing about KDE: with the key unset nothing runs, and each machine names its own script (OSL-1046 runs Plasma 5).
- Lym runs the command only once the new window is mapped, or after a short wait, so the script sees it.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - opentask-and-failed-launch-untested (class scope, severity MEDIUM, open in rounds 1)
  - prime-spawn-window-dir-is-file (class design, severity BLOCKING, open in rounds 1)
  - window-command-timeout-untestable (class design, severity MEDIUM, open in rounds 1)
  - window-wait-unpinned (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
