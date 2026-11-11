```yaml
lens: board-scope
outcome: reported
```

## Findings

None. No contradiction with the board.

## Goal points of the entry

- Accounts in the claude config, each with name, config dir, weight and limit windows, machine-local — met (D1, machine layer).
- Weight sets the long-run share — met (D3 count-over-weight).
- Pick reads headroom in every window the account has, passes over one near a limit, never judges by a five-hour window it lacks — met (D1 windows, D3, D5).
- Pick reads both session and week figures (operator, 2026-11-11) — met (D5, status line `rate_limits`).
- Check what Claude Code exposes before choosing the headroom source — met (D5 facts with sources).
- Verify `CLAUDE_CONFIG_DIR` against the Claude Code docs — met (D4 facts with sources).
- Task states how the second account is wired for the test — met (D9, in the record, since the run may not edit its own entry).
- Heddle `claude-settings` is no source; no account detail in a GitHub-pushed file — met (D1 refuses tracked-layer value).
- Per-worktree pick, at first session, recorded so resume/re-attach/hand pass keep it — met (D2).
- Rule for prime's and beams' orch and dobby — met (D7).
- One place in claude engine's launch; precedence over the `env` module decided — met (D4).
- Usage limit: move vs report — met (D8, report and wait).
- `lym report` / `tokencount` read both accounts — met (D6).
- Not in scope: logging in, terms — honoured (D9 leaves login to the operator).

## Entries, peers, parent commits checked

- `per-machine-branch-prefix` (F4): no conflict — parked note on per-machine keys; D1's machine-layer use is consistent with it.
- `sandbox-closed-area-build` (F9): no conflict — only mentions claude.com hosts for a proxy.
- `codex-engine-trial` (P2): no conflict — board weighs the two-account route as first; Codex stays out of scope.
- Peers `parent-reviews-decision-record`, `hub-workspace-all-pairs`, `testmap-selector-upkeep-rules`: no conflict — changed paths don't touch the claude engine, claude config or reed launch.
- Parent commits: `lym loom peers` lists none.

## Decision notes

- No `Decision for claude-account-weights:` note on the board; no `Added after Discussion` entries in the record. None unreachable.

Tool calls:
- `lym board get` for `per-machine-branch-prefix` and grep of `sandbox-closed-area-build` for `.claude` — my context held only their one-line briefs, and both touch config or Claude.
