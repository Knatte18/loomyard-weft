# Decision record — solo Codex Astra at high effort on P4

## Goal

Measure how much effort alone changes a solo Codex Astra plan review.
One `gpt-6-astra` review of `P4` at `model_reasoning_effort="high"`, graded against the Claude fan's findings and reported beside the medium solo row and the Claude fan row.

## Scope

- In: `docs/research/codex-astra-solo-high-bench-2026-10-27/report.md` and its `data/`: `inputs.txt`, `usage.txt`, `match.tsv`, `reviews/P4/`, `harness/`.
- Input: `P4` (`machine-handover`), rebuilt from the solo bench's weft commit and code commit.
- One `codex exec` call, no repeat, no Claude run.
- Out: no change to Go code, `PATTERN.md`, `doc.go` files or `docs/overview.md`; the bench is shell scripts, the reviewer's output and a hand grading.

## Decisions

### Harness copied from the earlier benches, paths re-pointed, effort changed

Decision: `data/harness/` holds the solo bench's `build.sh`, `run.sh`, `setup-codex-home.sh`, `windows.sh` and `dump.sh`.
The only content changes are the `ROOT` and `WEFT` paths, the binary name `lyx` to `lym` in the stencil paths where the snapshot needs it, and `model_reasoning_effort="high"`.
Rationale: the task asks for the solo harness unchanged except for the effort; the paths point at a different worktree and cannot stay.
The Opus script is left out, since no Claude run is made.

### Prompt and snapshot equal the solo bench's

Decision: `build.sh` rebuilds `P4` from weft commit `045707759` and code commit `750903aef1b7474b06597cdd2d6e0aecfd85d251`, the values in the solo bench's `data/inputs.txt`.
Rationale: only the effort may differ between the medium and high rows, so the prompt, snapshot and sandbox flags stay identical.

### Windows read after the call, stop rule applies to the reading

Decision: the rate-limit windows are read from the last `token_count` event of the call's rollout.
No call is made before the reading exists, so the first reading follows the one call.
The 70 percent stop is checked on that reading, and any window above it is reported as the stop.
Bound: the cap cannot prevent the one call's cost; it limits the report to what was done and forbids any further call.

### Grading in the earlier benches' way

Decision: each finding is labelled `shared`, `unique-real` or `false` against the fan's round-1 findings of `P4` (`claude-findings.tsv` of the first bench) and, when unmatched, against the cited card lines and the snapshot code.
`data/match.tsv` carries the label, the reviewer's severity and the fan finding matched.
Rationale: the report places the row beside the medium solo and fan rows, so the labels must mean the same.

### Medium solo and fan rows are cited, not re-run

Decision: the medium solo row and the Claude fan row come from the solo bench's `usage.txt`, `match.tsv` and report.
Rationale: the task caps the work at one call.

## Constraints

- `CLAUDE.md`: agents run in interactive tmux, never `claude -p`; the bench makes no Claude run.
- `CLAUDE.md`: markdown uses semantic line breaks; table cells stay on one line.
- `PATTERN-lym-temp-root` governs `lym` code only; bench scratch lives under the worktree's `.scratch/`, which is not committed.
- No production deploy, no push, no edit outside this worktree.
- Stop and report what was done if a rate-limit window passes 70 percent.

## Auto-mode assumptions

- The `codex` CLI and the operator's `~/.codex/auth.json` are usable from this session; the scratch `CODEX_HOME` links to `auth.json` as the earlier benches did.
- The weft repository at `/home/knatte/Code/loomyard-LYMHUB/loomyard-weft` still holds weft commit `045707759`.
- A snapshot older than the `lyx` to `lym` rename may need the stencil paths read under their old names; the build script copies them by pattern.

## Open risks

- One input, one run: no variance is measured, and a repeat could change a count by one or two.
- The grader is a Claude session and may favour its own family's phrasing.
- The plan tier may report only a weekly window, so the 5-hour points of the solo bench may not be comparable.
- The operator's `~/.codex/config.toml` may influence the run and is not hashed.

## Acceptance criteria

- `report.md` exists with a conclusion, a summary table with the high-effort row beside the medium solo and Claude fan rows, per-finding detail, method, deviations and a data list.
- The report records wall time, fresh input, output and reasoning tokens, and the windows for the call.
- `data/match.tsv` grades every finding of the high review as `shared`, `unique-real` or `false`.
- `data/reviews/P4/` holds the review file; `data/usage.txt` holds the call's line.
- Exactly one `codex exec` call was made.
- `lym loom validate-description` passes on the change description.
