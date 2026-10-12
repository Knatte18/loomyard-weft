# Fixer report — plan, round 1

## Changes

- F1 (fixed as proposed): card 5's Intent gains a bullet after the two guard skips saying both let through exactly the root children in `shortcutcli.GroupID`, bounded by the shortcut file.
  `TestShortcuts_FileAndRootListing` gains an assertion that the names of the root's children in `shortcutcli.GroupID` equal, as a set, the names `shortcutcli.Load()` returns.
- F2 (fixed as proposed): card 5's Edit target `cmd/lyx#newRoot` becomes `cmd/lyx/main.go`, and card 6's `internal/loomcli#statusStrandShareDen` becomes `internal/loomcli/bootstrap.go`.
  `cmd/lyx/main.go:1-12` holds the file and package comments, and `internal/loomcli/bootstrap.go:25` holds the const-block comment.
- F3 (fixed as proposed): card 5's root Long line becomes "Shortcuts, declared in internal/shortcutcli's embedded file, are listed under their own heading in lyx --help.", with no shortcut names.
- F4 (fixed as proposed): `TestShortcuts_FileAndRootListing`'s synthetic rows gain the argv `[--verbose]`, which expects the problem naming the root.
  The root declares `--verbose` as a persistent count flag (`cmd/lyx/main.go:121`), so `root.Find` strips it and resolves to the root with no error.

## Disputed

None.

## Deferred

None.
