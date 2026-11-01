---
format: 5
approved: false
language: go
---

# Plan: machine-switch — `lym adopt` takes a beam or root over and leaves every run in a known state

`lym adopt` already reconciles origin, settles local commits, adds pairs, writes owners and brings up each beam's dobby and orch.
This plan makes the runs it takes over wait for their subject's adopt check: adopt holds them with a new pause caller, runs one check per subject in that subject's worktree, a green check lifts the holds and starts the runs, and a red one files a darn fix task through a new general front-task piece, which dobby starts ahead of the worktree's other tasks and whose landing checks again, up to `adopt_fix_tries`.
The darn writer gains a stop on a condition its task body names, for a machine failure, and adopt reports one line per run.

## Card Index

1 — hold-caller — the pause record's `HoldCaller` and `Coverage.Held`
2 — dobby-hold — dobby gives a held run no pass and owes it no lift pass
3 — board-front-fields — the board's `front` and `on_land` fields, written only by lym, with fixed project, kind and slug
4 — batten-cancel-export — an in-process batten cancel whose nothing-left refusal is matchable
5 — adopt-check-per-subject — one adopt check per subject, in its worktree, with per-subject files and view
6 — front-task-package — `internal/fronttask`: file, remove and settle a front task and its land action
7 — dobby-front-starts — dobby starts an open front task first, past `max_runs`, and settles its action
8 — darn-condition-stop — the darn writer stops on a condition its task body names
9 — adopt-fix-tries-key — beam.yaml's `adopt_fix_tries`
10 — task-branch-slugs — fabric lists the task branches of a slug prefix
11 — adopt-holds — adopt holds the runs it takes over before the owner write, and resumes none
12 — check-green-end — a green check lifts its subject's holds, starts the runs and removes the fix task
13 — check-fix-loop — a failed check files the subject's fix task, up to the try limit
14 — adopt-run-report — adopt reports one line per run

## Shared Decisions

- The hold is a pause-record entry of `pauserecord.HoldCaller`; nothing but the pause record gates a held run, so dobby, batten passes and the network gate need no new gate (D4).
- Per-subject state sits in `<AdoptScratchDir>/<subject>/`: `check.json`, `check.lock`, `check.out`, `suite.log`, `holds.json` and `fix-tries.json`, each written only by `adoptcli`, the last two never by the check record's writes.
- `internal/fronttask` holds everything of the front task but the board fields and dobby's start hold; dobbyengine judges an open front task from its `TaskView` and calls `fronttask.Settle` through one seam, so the told-geometry engine imports no new package.
- The check's green and failed ends are seams of `runCheck` filled by `hubCheckDeps`, so `TestRunCheck` keeps faking the gate and the end logic has tests of its own over fakes.

## PATTERN changes

- `PATTERN-adopt-hold` — adds: On a takeover, `lym adopt` holds it, and each unstarted released task of the subject bar a front task, with a `pauserecord` hold-caller entry; only the green adopt check of the subject lifts it, never dobby; by hand, `lym tender resume` or `lym batten cancel`.
- `PATTERN-front-task` — adds: Only fronttask files a front task and writes `front` and `on_land`; while one is open (empty or run status) dobby starts no other task in its worktree, and runs its `on_land` action from fronttask's closed table, on the subject's owner only; its project, kind and slug are fixed.
