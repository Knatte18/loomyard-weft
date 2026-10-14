## Changes

- F2: `docs/overview.md` batten entry no longer says `Run-Shed` opens the window or keeps a once-marker.
  It now says the `Open-Window`, `Seed-IDE` and `Open-IDE` rows own the window and VS Code, and that a run resumed past `Open-Window` opens no window.
  The window-closing paragraph is unchanged.
- F3: `internal/battencli/cli.go` `terminal_window` text now reads "opens through <launcher> at the Open-Window row".
  No test or doc asserted the old wording.
- F4: `checkTaskSlugOpensBareFolder` in `internal/ideengine/spawn_hub_integration_test.go` now removes the keybindings file, calls `SeedTask` and asserts no file at the `KeybindingsPath` seam, then calls `Spawn` and asserts the file exists.
  No new top-level test.
- `lyx gate test --tags integration` is green for `./internal/ideengine` and `./internal/battencli`.

## Disputed

None.

## Deferred

- F1: the row placement conflicts with the board entry's scope item 2.
  Settling it takes `lyx loom decision add`, which is the run's parent's call and outside the fixer's write surface.
  The code is left in place, as the finding's `fix` says.
