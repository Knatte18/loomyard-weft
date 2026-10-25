```yaml
lens: code-pattern-fit
outcome: reported
```

Lens code-pattern-fit: the diff checked against CLAUDE.md and PATTERN.md. I made 3 tool calls.

**Rules checked**

| Rule | Result |
|---|---|
| Diff under `docs/research/codex-review-bench-2026-10-25/` only (decision-record Constraints) | Held. `git diff --stat` lists only that directory, with no `.scratch` content. |
| Docs-only commit, no build or test reads it | Held. `internal/lyxcwd/enforcement_test.go:565` lists `docs/research/` in `vocabScanAllowlist`, so the fabric-vocabulary scan skips it. |
| `PATTERN-fabric-vocabulary` | Held. `docs/research/` is allowlisted, so the bare weft/warp wording in the report is not scanned. |
| `PATTERN-durable-vs-ephemeral-state` | Held. The scripts name `.scratch/`, which is neither `.lyx` nor `_lyx`. |
| `PATTERN-lyx-temp-root` | Held. The scripts use `.scratch/`, never `os.TempDir()`. |
| `PATTERN-github-auth` | Held. No script runs `gh`. |
| `PATTERN-spawn-observability` and the other agent PATTERN entries | Not applicable. `lyx` spawns nothing here. |
| Agents run in interactive tmux, never `claude -p` | Held. `codex exec` is not a `lyx` agent spawn. |
| Declared PATTERN changes landed | Held. The directive declares none, and `PATTERN.md` and `pattern/` are untouched. |
| Commit message and `Pattern-Departure` trailer | Held. The one commit carries the `Co-Authored-By` line, and no departure is claimed. |
| Docs-in-same-commit | Not applicable. No module or CLI change. |
| No perishable tallies in prose | Held. The tallies sit in a dated measurement report, which is the stated exception. |
| Table cells on one line | Held. Every table row in `report.md` is one line. |
| Markdown semantic line breaks (CLAUDE.md "Markdown") | **Broken**, see the finding below. |

**Findings**

**LOW, consistency**: `report.md` breaks the semantic-line-break rule.
- Location: `docs/research/codex-review-bench-2026-10-25/report.md`.
- Each line below holds two independent clauses that are joined by a semicolon or by a comma and a conjunction, and the rule puts a break between them. Evidence is the lines as Read:
  - Line 9: `..., and the bench stopped early...`
  - Line 16: `..., and the limit stopped it before the plans`
  - Line 40: `...; cached reads are...`
  - Line 41: `..., so a single run is known...`
  - Line 42: `...; Claude approved only D3...`
  - Line 64: `...; each snapshot is...`
  - Line 66: `...; each snapshot is...`
  - Line 86: `..., and the bench's sessions...`
  - Line 88: `...; every run...`
  - Line 96: `...; Luna's D4 F1...`
  - Line 129: `...; the closing turn.completed...`
  - Line 132: `...; no trust prompt...`
  - Line 133: `...; the largest plan...`
  - Line 147: `..., and the four Astra plan runs...`
  - Line 151: `..., and the bench stopped only...`
  - Line 155: `...; Astra was the part...`
  - Lines 159, 162 and 163: each has a semicolon between clauses.
- Fix: break each of these lines at the clause boundary, so the new line starts at the second clause.

**LOW, consistency**: the Codex review files under `data/reviews/*/*.md` may break the same rule. The grep counted 189 lines over 200 characters across them. I did not read these files, so this is unverified.
- Location: `docs/research/codex-review-bench-2026-10-25/data/reviews/`.
- These files are raw Codex output kept as measurement data. If the handler treats them as verbatim data, the finding does not apply.
- Fix: leave them as recorded, or note in `report.md` that `data/reviews/` is verbatim.

**Tool calls**
1. `grep -rln "docs/research"` over `.go`, `.yaml` and `.sh` files, plus an `awk` count of review-file lines over 200 characters. I needed to know whether any test or config reads `docs/research`.
2. `grep -n "docs/research"` in `internal/lyxcwd/enforcement_test.go` and `internal/lenses/config_test.go`. This narrowed the hits to the one that applies a rule.
3. `sed -n 540,600p internal/lyxcwd/enforcement_test.go`. This confirmed the allowlist entry for `docs/research/`.
