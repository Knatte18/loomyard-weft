# PATTERN entries and scans for interactive spawns and filesystem links

Two rules that stood only in the loomyard `CLAUDE.md` become PATTERN entries, each held by a tier0 guard scan in `cmd/lym`, so they survive that file's deletion in the follow-up task `claude-md-lym-rules-out`.
`CLAUDE.md` itself is unchanged.

## What changes

- `PATTERN-interactive-spawn` (under "Agents and prompts"): spawning an agent runs it as an interactive tmux session driven through reed, never headless `claude` with `-p` or `--print`.
  `cmd/lym/interactivespawn_test.go` parses every production Go file in the module, `tools/` included, and reports three shapes:
  a call that takes a print-flag string literal and another argument naming claude (a `claude`/`claude.exe` literal, or an identifier, selector or call whose name holds `claude`);
  a string literal whose tokens name claude and then carry a print flag;
  and any string literal under `internal/shuttleengine/claudeengine` with a print-flag token, since that engine builds launch lines from flag fragments.
- `PATTERN-fs-links` (under "Paths and geometry"): a filesystem link is created only through `internal/fslink`'s `CreateDirLink`.
  `cmd/lym/fslinkonly_test.go` reports every `os.Symlink` or `os.Link` call outside `internal/fslink` in production Go files, through any name the file binds `os` to.
- Each entry carries `(test)` and a background file under `pattern/` that sets out the scan's scope and bounds.
- `cmd/lym/tierpurity_test.go` gets an `allowedSpawners` entry for the new spawn scan, whose fixture sources contain `exec.Command` as text to parse, as the other guard scans' fixtures do.

## Design decisions

- **Production files only.** Neither scan reads test files.
  Tests across fabricengine and other packages plant symlinks as hostile fixtures that fslink's refusals would block, and the spawn scan's own fixtures hold the forbidden text as data.
- **One allowlisted link site.** `tools/deploy/main.go`'s `copyEntry` copies a plugin-tree symlink verbatim.
  The link may point at a file, and fslink has no file-link primitive, so the site is admitted with that reason rather than rerouted.
- **The links entry makes no Windows claim.** It states only what the scan holds.
  Its background file points at `internal/fabricengine`'s package doc, which records that fslink's Windows junction path has never been run.
- **AST, not substring.** Both scans parse files, so the doc comments and README text that name `claude -p` never count.

## Bounds

- The spawn scan sees static shapes only: a flag built at runtime, or a launch line assembled from fragments outside the claude engine's directory, passes.
- The spawn scan's identifier heuristic could flag an unrelated call that takes a `-p` literal beside a name holding `claude`; the remedy is an allowlist entry with a reason.

## Tests

Each scan has a real-tree guard test (`//lym:guard`) and a table of fixture trees.
The tables cover each violation shape being found, and clean cases passing: an interactive launch, tmux's `-p`, a comment, a `-p` fragment outside the claude engine, a call inside `internal/fslink`, `os.Readlink` and a test file.
