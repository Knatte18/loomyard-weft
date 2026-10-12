# Round 1 fixer-report

## Changes

- F1 (LOW), fixed as proposed:
  `internal/shortcutcli/doc.go` now says that only a word which raises help prints the shortcut's help, and that a false-valued one (`--json=false`, `--help=false`) is dropped and the argvs run.
  `TestShortcut_Run` gained one row, `prime --json=false`, expecting both runs and exit 0.
  `lyx gate test ./internal/shortcutcli` passes.

## Disputed

None.

## Deferred

None.
