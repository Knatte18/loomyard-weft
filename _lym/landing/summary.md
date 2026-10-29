# Add PATTERN entries and scans for interactive spawns and fs links

Two rules that stood only in `CLAUDE.md` become PATTERN entries, each held by a scan in `cmd/lym`, so they survive that file's deletion.
`CLAUDE.md` is untouched; its removal is the follow-up task `claude-md-lym-rules-out`.

## What changes

- `PATTERN-interactive-spawn` (group "Agents and prompts", `(test)`): an agent runs as an interactive tmux session driven through reed, never headless `claude` with `-p` or `--print`.
  The reason, subscription coverage versus API billing, is in `pattern/PATTERN-interactive-spawn.md`.
- `PATTERN-fs-links` (group "Paths and geometry", `(test)`): a link is created only through `internal/fslink`'s `CreateDirLink`, with no `os.Symlink` or `os.Link` outside `internal/fslink`.
  The entry makes no Windows claim; the background file points at `internal/fabricengine`'s package documentation, where that stays unsettled.
- `cmd/lym/interactivespawn_test.go` and `cmd/lym/fslinkonly_test.go` are the scans, each with fixture cases of a violation found and of clean input passing.
- `cmd/lym/tierpurity_test.go` allowlists the spawn scan's test file, whose fixtures hold `exec.Command` as source text that is only parsed.

## Design decisions

- Both scans parse every production Go file of the module, `tools/` included, and read no comments, so docs naming `claude -p` pass.
  Test files are not scanned: a test may hold the forbidden text as data, and tests plant links as hostile or foreign fixtures that fslink's refusals would block.
- The spawn scan reports three static shapes:
  a call with a print-flag literal and another argument naming claude;
  a string literal whose tokens name claude and then carry a print flag;
  any print-flag token in a string literal under `internal/shuttleengine/claudeengine`, which builds launch lines from flag fragments.
  Its allowlist is keyed by file, is empty, and reports stale entries; it is consulted only for a file that has findings.
- The links scan reports calls through any name the file binds `os` to, and reports a dot import of `os` at the import line, since that binds no name.
  Its allowlist is keyed `<file>:<call>` and holds one entry: `tools/deploy/main.go`'s `copyEntry` copies a plugin-tree symlink verbatim, the link may point at a file, and fslink has no file-link primitive.
- Bounds: the spawn scan misses a flag built at runtime or a launch line assembled outside the claude engine's directory, and the links scan misses `os.Symlink` taken as a function value.
  The identifier heuristic (a name holding `claude`) may flag an unrelated call carrying a `-p` literal; an allowlist entry with a reason is the remedy.

## To check by hand

- No whole-module run was made here; run `lym gate test --upto 2` on `main` before pushing.
- Dot import of `os` in `cmd/lym/fslinkonly_test.go:46`: confirm an `import . "os"` file with a bare `Symlink` call is reported and an `import _ "os"` file is not (a fixture row covers each).
- Allowlist order in `cmd/lym/interactivespawn_test.go:114`: confirm an allowlist entry for a file with no remaining finding is reported stale, and one for a violating file admits it (`TestInteractiveSpawn_Allowlist`).
