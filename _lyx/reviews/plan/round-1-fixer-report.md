# Plan round 1 — fixer report

## Changes

- **F1** (MEDIUM), fixed as proposed in `_lyx/plan/06-config-terminal-guard.md`.
  The premise holds: the subtest at `internal/configcli/configcli_integration_test.go:188` runs `menu` through `Command()` over a `strings.Reader`, and `menu`'s new terminal check would refuse it before the listing and the hub-write guard.
  Card 6's Intent now says that subtest calls `menu` directly with `terminal` true, over `dirsOf` of the code worktree's resolved Location with `dirs.hubWriteGuard` set to `realHubWriteGuard`, and keeps its assertions.
  It also names the inputs a direct call needs and the old one got from the command: the board choice, a never-called editor, and no sync or commit.
- **F2** (LOW), fixed as proposed in `_lyx/plan/05-config-empty-edit-abort.md`.
  The premise holds: `EditPath`'s unchanged-unparseable branch returns `ErrAborted` without a restore, and `TestEdit`'s row "unchanged invalid YAML aborts and keeps a pre-existing file" pins `wantFile: ptr(invalid)`.
  The card now limits the comment's restore to the empty-result abort and says the other abort paths remove a scaffolded file and leave a pre-existing file as the editor left it.
- **F3** (NIT), fixed as proposed in `_lyx/plan/05-config-empty-edit-abort.md`.
  The premise holds: `TestEditOne` calls only `editOne(worktreeDirs(baseDir), ...)`.
  The card now credits only `editOne`'s arm to that row and names `editLocal`'s arm as left uncovered, since this card does not change how it reports.

## Disputed

None.

## Deferred

None.
