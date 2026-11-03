# Preflight blocks on lym's own untracked go-settle.jsonl under _lym (same pattern as #22)

`preflight-clean-skips-lym-state` · Note · bug, preflight, shed

Imported from [issue #27](https://github.com/Knatte18/loomyard/issues/27).

## What happened

Run `commonlib-fluid-report-selection` (tapestry, beam `duplication-and-import-followups`, Models hub) blocked at Preflight with:

```
preconditions not met: worktree-clean: uncommitted state changes under `_lym`: ?? src/csharp/NORCE.Models/_lym/shed/commonlib-fluid-report-selection/go-settle.jsonl
```

`go-settle.jsonl` is lym's own settle log.
The tackler spawned on the Preflight stop wrote it, left it untracked in the weft, then handed the stop back; Preflight's worktree-clean check then refuses that same file on every re-step.
The worktree had no code changes.

Log content at the stop:

```
{"stop_key":{"current_producer":"Preflight","history_length":1,"state":"blocked"},"settler":"tackler","action":"spawn","outcome":"tackler spawned",...}
{"stop_key":{"current_producer":"Preflight","history_length":1,"state":"blocked"},"settler":"tackler","action":"restep","outcome":"tackler resume",...}
{"stop_key":{"current_producer":"Preflight","history_length":3,"state":"blocked"},"settler":"tackler","action":"spawn","outcome":"tackler spawned",...}
{"stop_key":{"current_producer":"Preflight","history_length":3,"state":"blocked"},"settler":"parent","action":"hand-back","outcome":"the tackler handed the stop back",...}
```

The run had stopped once before at Seed-Child (hub profiles.yaml lacked tapestry blocks) and was re-run with `lym batten run`.

## Same pattern as #22

#22 is the halt marker left uncommitted under `_lym`, which Preflight then refuses.
This is a second file of the same kind: state lym writes into `_lym` while handling a stop is neither committed nor exempt from worktree-clean, so lym blocks on its own bookkeeping.
A fix for one file name leaves the pattern open; every file lym writes under `_lym` on a stop path needs to be committed when written, or excluded from the clean check.

## Workaround

From the task worktree: `lym fabric commit`, then `lym loom resume` (run by the orch).
The run then went on to Discussion-Write.