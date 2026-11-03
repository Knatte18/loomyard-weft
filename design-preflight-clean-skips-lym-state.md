# Preflight refuses lym own state files under _lym

`preflight-clean-skips-lym-state` · Task · bug, preflight, shed · running · Code-Burler-Review

From [issue #27](https://github.com/Knatte18/loomyard/issues/27).

## Seen

Run `commonlib-fluid-report-selection` (tapestry, Models hub, beam `duplication-and-import-followups`) blocked at Preflight:
"preconditions not met: worktree-clean: uncommitted state changes under `_lym`: ?? src/csharp/NORCE.Models/_lym/shed/commonlib-fluid-report-selection/go-settle.jsonl".

`go-settle.jsonl` is lym's own settle log.
The tackler spawned on an earlier Preflight stop wrote it and left it untracked in the weft, so Preflight's worktree-clean check refused that same file on every re-step until the tackler handed the stop back.
The worktree had no code changes.

Workaround: `lym fabric commit` in the task worktree, then `lym loom resume`.

## The pattern

This is the second file of one kind: issue #22 was the inherited halt marker under `_lym`, and this is the settle log.
State lym writes under `_lym` while handling a stop is neither committed when written nor exempt from worktree-clean, so lym blocks on its own bookkeeping.

## Expected

- Fix the pattern, not the one file name: every file lym writes under `_lym` on a stop or settle path is either committed when written or excluded from Preflight's worktree-clean check, decided once in one place.
- A test walks the files the stop and settle paths write under `_lym` and proves none of them makes worktree-clean refuse.
