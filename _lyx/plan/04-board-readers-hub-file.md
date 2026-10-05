# Card 4 — Every board-config reader loads the hub's board.yaml

**Edit:**
- `internal/boardengine#LoadConfig`
- `internal/boardengine/config.go`
- `internal/boardengine/config_test.go`
- `internal/boardengine/doc.go`
- `internal/boardcli/cli.go`
- `internal/boardcli/doc.go`
- `internal/boardcli/cli_test.go`
- `internal/boardcli/verbs_test.go`
- `internal/loomcli#openHubBoard`
- `internal/loomcli/landingdeps_integration_test.go`
- `internal/ideengine#Menu`
- `internal/ideengine/menu_test.go`
- `cmd/lyx/main_integration_test.go`
- `plugins/ly/skills/board/SKILL.md`

**Uses:**
- `internal/fabricengine#BoardDir`
- `internal/configengine#ConfigFile`
- `internal/configengine#ErrNotInitialized`

**Intent:**
Every reader of board config passes `fabricengine.BoardDir(hub)` to `boardengine.LoadConfig` instead of a worktree dir (D3); `LoadConfig` keeps its told-`baseDir` signature (`PATTERN-told-geometry`) and its strict `configengine.Load` (`PATTERN-config-strictness`).
- `boardcli`'s `PersistentPreRunE` resolves the layout with `lyxcwd.Resolve(cwd)` first, then loads from `fabricengine.BoardDir(layout.HubPath)` and sets `cfg.Path` from the same value.
  The `--board-path` branch for the detached sync child is unchanged and loads no config.
- `loomcli`'s `openHubBoard` and `ideengine.Menu` load from `fabricengine.BoardDir(l.HubPath)`, the dir they already set as `Path`.

A per-worktree `board.yaml` alone no longer satisfies any of them.

`LoadConfig` today maps only an absent `_lyx/` dir (the "not initialized" error text) to the `lyx fabric reconcile` way forward.
An absent `board.yaml` inside a present `<BoardDir>/_lyx/` falls through to `configengine`'s "run lyx config reconcile", which is the wrong command now.
Make both absences, checked by `errors.Is(err, configengine.ErrNotInitialized)` or by the file at `configengine.ConfigFile(baseDir, module)` not existing, return `board config <path> not initialized; run "lyx fabric reconcile"`, naming the hub file.
The old message text `not initialized here; run "lyx fabric reconcile"` is asserted by no `boardengine` test (the hits in `landingshed` and `loomengine` are their own copies and stay).

Docs in this commit:
- `boardcli/doc.go` and `board`'s cobra `Long` say the config is the hub's `<hub>/_board/_lyx/config/board.yaml`, read whatever worktree the verb runs in.
- `boardengine/doc.go`'s label-list paragraph (the "write this list into the parent's live `_lyx/config/board.yaml`" note) points at the hub file.
- `Config.Path`'s comment in `config.go` stays accurate.
- `plugins/ly/skills/board/SKILL.md`'s sentence placing the label maps in `_lyx/config/board.yaml` names the hub's `board.yaml` instead; `lyx config board` stays the way to edit it.

Tests that seeded a worktree's `board.yaml` and then ran a board reader now seed `configengine.ConfigFile(fabricengine.BoardDir(<hub>), "board")` instead:
- `seedCwd` and the `--board-path` test's seeding in `boardcli/cli_test.go`, where the hub is the temp dir's parent;
- the `board.yaml` rewrites in `verbs_test.go`;
- `TestRunDispatchesToBoard` in `cmd/lyx/main_integration_test.go`;
- the board config in `ideengine/menu_test.go`, whose board dir is already `<container>/_board`;
- `landingdeps_integration_test.go`'s `LoadConfig` call, which reads the hub file a hubforge hub now carries from card 3's clone reconcile.

Coverage:
- `boardcli/cli_test.go` (integration) gains one test for the acceptance criteria.
  A hub `board.yaml` declares a type `feature` and a label the worktree copy lacks, and the worktree's own `board.yaml` lacks both: an upsert using `feature` and that label succeeds, and the rendered README has a `feature` section.
  A second row deletes the hub file and keeps only the worktree copy: the verb exits 1 with an error naming `lyx fabric reconcile`.
- `boardengine/config_test.go`: one table-driven test with rows for an absent `_lyx/` and for an absent `board.yaml` under a present `_lyx/config/`, each asserting the error names the file path and `lyx fabric reconcile`.

**ImpactSummary:** Three board-config call sites switch base dir and `LoadConfig`'s absence message changes; tests that seeded a per-worktree `board.yaml` now seed the hub file.

**Commit:** 4: board: load board.yaml from the hub's board dir
