```yaml
lens: code-pattern-fit
outcome: reported
```

Task: check the one commit (report plus two TSV logs) against the rules in `CLAUDE.md` and PATTERN, which the diff does not touch.

**Rules checked**
- Scope, "No file outside `docs/research/creel-hook-delivery-2026-10-25/` and `.scratch/` changes": held. The diff stat shows only the report and the two TSVs under that directory.
- `CLAUDE.md` worktrees (own worktree only, no push, no `update-plugins.sh`): held. The commit is on the task branch, and the report says the prototype stayed under this worktree's `.scratch/`.
- No hook in user or plugin settings: held. The hooks were given per session by `--settings` and `-c` flags. The `~/.claude.json` and `~/.codex/config.toml` trust entries are not hooks, and the report names both.
- Docs land in the same commit (`doc.go`, `docs/overview.md`, PATTERN): not applicable. The diff has no module, CLI or cross-cutting change.
- Declared PATTERN changes: none are declared and the diff has no `PATTERN.md` or `pattern/` change, so nothing is owed.
- PATTERN entries: none apply, because the diff has no Go code, temp files, git calls, stencils or spawns.
- Commit message and departures: held. There is no `Pattern-Departure:` trailer and none is needed. The `Co-Authored-By` trailer is present.
- Markdown, one sentence per line, table cells on one line: partly broken, see F1 and F2. Every table cell is on one line.
- Notes go in git: held. The report is committed and does not depend on `memory/`.

**Findings**

F1 — NIT, consistency — `docs/research/creel-hook-delivery-2026-10-25/report.md:92`
- Line 92, "It also tried to answer the idle-path message; …", has no indent under the list item that starts on line 91.
- The sibling continuation lines, such as line 93, are indented two spaces.
- Markdown reads it as a lazy continuation, so it renders inside the item, but the file is inconsistent and the list structure is fragile.
- Evidence: report.md lines 91–93.

F2 — NIT, consistency — `docs/research/creel-hook-delivery-2026-10-25/report.md:61, 83, 100, 101, 115, 122, 126, 130`
- These lines hold two independent clauses on one line, joined by a semicolon or by a comma plus "and" or "but", each clause with its own subject and verb.
- Examples:
  - Line 83: "…the 1.8 s gap … is within the model's own latency spread across single samples, and one sample of SendMessage cannot settle which is faster."
  - Line 101: "Codex started with … and accepted the config, but the user `!` shell command fired neither hook."
  - Line 130: "the hook matches SendMessage in kind, and one pair of samples cannot rank them."
- `CLAUDE.md` points to `scribe:prose`, which breaks such lines at the clause boundary.
- Each line is still a single sentence, so `CLAUDE.md`'s own "one sentence per line" test is met.
- Evidence: report.md lines 61, 83, 100, 101, 115, 122, 126, 130.

**Tool calls:** none. The diff, report, logs, summary, rubric and `CLAUDE.md` rules were already in my context.
