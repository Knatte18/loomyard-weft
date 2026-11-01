# Delete the one-shot boltrename program

`tools/boltrename` moved existing hubs from the `_board` checkout directory to `_bolt`.
Neve is already migrated and OSL-1046 is re-cloned rather than migrated, so no hub is left for it to run on.

This removes the program, its tests and its test main (`main.go`, `rename_test.go`, `testmain_test.go`).
No other file mentioned it, so nothing else changes and no refactor rides along.

## Check

A repository-wide search for `boltrename` finds nothing outside git history.
