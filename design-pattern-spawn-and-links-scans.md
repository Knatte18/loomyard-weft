# PATTERN entries and scans for interactive spawns and filesystem links

`pattern-spawn-and-links-scans` · Task · enhancement, pattern, shuttle, fabric · running · Drift-Baseline

## Goal

Two rules that stand today only in the loomyard `CLAUDE.md` become PATTERN entries with a scan each, so they survive the file's deletion (`docs/research/claude-md-rule-map-2026-10-29.md`, operator decision 2026-10-29).

## Scope

- A PATTERN entry for spawning an agent: every agent lym spawns runs as an interactive tmux session driven through reed, never `claude -p`, with a scan that fails on a `claude` spawn carrying `-p`.
  Read `README.md` and `internal/shuttleengine/doc.go` for the rule's current wording, and `internal/pattern` for how an entry and its scan are written.
- A PATTERN entry for filesystem links: every link goes through `internal/fslink` (`CreateDirLink`), directory junctions on Windows and symlinks elsewhere, with a scan that fails on `os.Symlink` or `os.Link` outside `internal/fslink`.
  `internal/fabricengine/doc.go` notes that the Windows claim of the rule is unsettled; the entry states only what the scan can hold.
- Each entry carries its `(test)` mark, and the change adds the scans' tests, with a case of the scan finding a violation and a case of it passing.
- No change to `CLAUDE.md`; its deletion is the task that follows (`claude-md-lym-rules-out`).
