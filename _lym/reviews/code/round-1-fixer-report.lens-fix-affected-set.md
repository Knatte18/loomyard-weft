```yaml
lens: fix-affected-set
outcome: reported
```

No findings. I checked F2 and F3, the findings whose premise is a set of sites. F1 is a single site.

- **F2 (every window reed opens):** I searched non-test Go under `internal` for `new-window`, `break-pane` and `new-session`.
  - `new-session` is at `lifecycle.go:624` only, built by `serverSpawnArgv`. Its one spawn at `lifecycle.go:469` flows into `logBootWindowLocked` at `:545`, which is the boot's first-window Info line.
  - `new-window` goes through `newWindowLocked` (`spawn.go:254` and `:258`), reached from the own-window path and from `window.go`. `spawn.go:328` logs the own window, and `window.go:125` logs the detached window with window id, name and command, and no pane id.
  - `break-pane` is at `ownwindow.go:425` and logged at `:439`.
  - `probe.go:33` and `:36` only list verbs and do not open windows.
  - The reworded `doc.go` lines 663–667 match that set: four sites, with "pane id where the open reports one" covering the detached line. No other `.go` or `.md` file outside `_lym` and `.lym` repeats the old "opened with a name" claim.
- **F3 (name field):**
  - The two `name` log sites are `spawn.go:328` and `ownwindow.go:439`. No test in the package references `openOwnWindowLocked` or `moveShellWindowStrandsLocked` or asserts these two log lines, so leaving the name field as log-only is the review's own fallback.
  - `ownwindow_integration_test.go` and `windoworder_integration_test.go` have no `logcapture`.
  - The log-only call is not a finding.

Tool calls:
1. grep of window-opening and Info-line sites across non-test Go: needed the full set of sites, which my context lacked.
2. grep of the doc claim wording across `.go` and `.md`: needed to find twin statements of the old claim.
3. grep for `serverSpawnArgv` and `logBootWindowLocked` callers, plus Info lines in `window.go`: needed to confirm no other boot or revival path skips the boot-window log, and that the detached-window line exists.
