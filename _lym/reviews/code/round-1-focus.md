---
round: 1
focus:
  - "`cmd/lym/interactivespawn_test.go`: do the three scan shapes match decision-record D2 (print-flag set, claude-naming rule for arguments, token rule, claude-engine directory rule), and does any shape miss a real spawn or flag an unrelated call on the current tree?"
  - "`cmd/lym/interactivespawn_test.go`: the allowlist is keyed `<file>` with a reason and reports stale entries; check it is wired to `scankit.NewAllowlist`/`RequireNoStale` as D2 asks and that the fixture cases cover each acceptance-criteria case (each of the three shapes, clean tree, comment, tmux's `-p`)."
  - "`cmd/lym/fslinkonly_test.go`: does the scan handle every way a file binds `os` (aliased, dot or blank import), and does `osImportName` exist and behave as the scan assumes; does the `<file>:<call>` allowlist for `tools/deploy/main.go` match D4?"
  - "`PATTERN.md` and `pattern/PATTERN-interactive-spawn.md`, `pattern/PATTERN-fs-links.md`: do the entries land in the groups and with the text, `(test)` mark and background link the decision record's PATTERN changes declare, and do the background files claim only what the scans hold (no Windows claim)?"
  - "`cmd/lym/tierpurity_test.go`: is the new `allowedSpawners` entry necessary and its reason accurate, and does it stay consistent with how the neighbouring entries are justified?"
  - "Test economy: do the new fixture tests duplicate a covering test, and does every behavior the diff adds have coverage?"
---

Seed focus from reading the answer key and the committed diff; the verify gate's subjects (build, tests, `pattern.Check`) are left to it.
