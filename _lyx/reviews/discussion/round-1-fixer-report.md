# Discussion round 1 — fixer report

All nine findings validated against the code and fixed; `lyx loom validate-discussion` exits 0 on the result.

## Fixed

- **F1 (D7).** Premise confirmed on a throwaway tmux 3.6 server: with `pane-border-status top` the window layout still sums to the full height, the top pane at `pane_top` 1 and one row shorter than its cell.
  D7 now keeps the status-row reservation as today, states that the layout sums to the window height, that the border title costs the top pane one content row, and how fixed-height pins treat it.
- **F2 (D12).** The ordered list now includes the booter at its own position; the booter brings up only earlier worktrees before creating its own session, and every nested up is told to skip the step, so it never recurses.
  The acceptance criterion now says sessions come back in spawn order, the booter's included.
- **F3 (D12).** The trigger is now a revival of the booter's own recorded session (whether or not the server answers), not a dead server, so a late booter fills earlier gaps first.
  Bound: never on a first boot, only worktrees whose reed state records a session (`Down` removes the state file, checked in `internal/reedengine/lifecycle.go`), no strand starts.
  Rationale and criterion updated; the Open risk now reads "revives".
- **F4 (D15).** Insertion moves to directly after the opening `[`, block entries each ending in a comma, so no outside byte changes.
  Added the JSONC scan rule (comments and strings skipped, markers as whole line comments), skips for no array, an unterminated, duplicated or inverted block, symlink handling, atomic temp-file-and-rename write, and no write on identical content.
  Criterion updated.
- **F5 (config colors).** The appended undated-id decision is removed and folded into D13 (config as the live source, the table as template defaults) and D14 (who reads it); Scope, Constraints (`PATTERN-config-strictness`), Auto-mode assumptions and the criterion now say the color matches the configured value, default D13's table.
- **F6 (bound and reader).** D13 states the closed palette, refusal at load of an out-of-palette value naming key, value and palette, and that only palette names reach a tmux format or a typed command.
  D14 names reed as the one config reader: it resolves the color when setting `@strand_color`, returns it on the added strand for shuttle, and the orch watcher takes the orch's color from reed.
- **F7 (provider seam).** D13's palette is now lyx's own vocabulary with no provider command or name; `claudeengine` maps palette colors to `/color` names, and the constraint says so.
- **F8 (pre-change panes).** The criterion now requires only `@strand` on a pre-change pane; a new Open risk says such a strand gets no tint until respawned.
- **F9 (support log).** `## Review rounds` records the parent change and this round; Q18 records the config-colors decision; the auto-mode line names D13's config colors as operator-sourced.
  Two rejected alternatives this round produced (dead-server-only trigger, insertion before `]`) are added to `## Rejected alternatives`.

## Disputed

None.

## Deferred

None.
