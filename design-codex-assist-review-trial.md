# Codex as assistant Code-Review reviewer on real darn runs

`codex-assist-review-trial` · Note · research, codex, shuttle

## Decision (operator, 2026-11-06)

Codex is used for review first, or as a support reviewer, not as a writer: the operator does not trust that it writes in the same tone as Claude.
The earlier idea of a writer comparison is dropped.

## Idea

`loom.yaml` already has `code_assist_reviewer_model`: an extra reviewer beside the Code-Review reviewer, whose report the reviewer consolidates and which never gates the round.
Set it per task through `run_settings`, as `{"loom":{"code_assist_reviewer_model":"astra[medium]"}}`, on real darn runs, and read afterwards what Codex found that the Claude reviewer did not, its false findings, and the time it added.

The benches `docs/research/codex/review/codex-code-review-bench-2026-10-28` and `codex-review-yardstick-2026-10-28` found Codex no stand-in for the Claude fan but with real findings of its own, which fits a support role.

## Results so far

Run 1, `begin-batch-wrong-conflict-kind` (`b93a0a52`), darn light, `astra[medium]` solo, no lenses, beside a `sonnet[medium]` Chair with three lenses, one round:

- Astra's one finding was real and no Claude seat had it: two test readers kept a literal beside the new constant, fixed in the landing.
  Astra filed it BLOCKING; the Chair kept it at LOW.
- Astra had no false finding, but missed both MEDIUM findings the affected-set lens found in the webster master stencil.
  It read the plan fixer's limit and judged the routing fine.
- Astra's review evidence was sound, but its sandbox could not run the gate: the runner lock under `_bolt/.lym/runproc/` was read-only.
- Time: Astra ran 11:22:27 to 11:24:36, in parallel; the Chair ended at 11:25:35, so at most a small wait on the wall clock.
  `round-<n>-usage.yaml` records no time or tokens for the assistant.

Run 2, part 1 of `codex-assist-fan-trial` (`2c4864bb`), one hand-composed `astra[medium]` run on `b93a0a52` with the three `review-code` lenses; report at `docs/research/codex/review/assist-fan-trial-2026-11-06/report.md`:

- All three lenses started and returned, and the review passed `lym burler validate-review`.
- Three MEDIUM findings, each checked against the tree and real, no false finding, 266 s, about 126,000 fresh input plus output tokens.
- Two wording points in `contracts/stencils/engine/codex/assist-fan.md`, neither blocking: it says `wait` where Astra used `wait_agent`, and "all in one go" where Astra spawned over 29 s.
  The run's change description says these were filed as notes; no such note exists, so they stand here.

Run 3, part 2 of the same task, the real composed seat beside a `sonnet[medium]` Chair with no lenses, on the report's diff, `review-code` fan asked for:

- The seat did not fan: no `spawn_agent` call, no lens report.
  It reported an instruction conflict and worked solo: the composed prompt told it to work alone and write its report "and nothing else", while the fan rules asked for lens reports.
- Two LOW findings, both real and both fixed in the landing, neither false; the Chair kept both and added a third LOW of its own.
- Time: 212 s from 13:00:44, in parallel; the Chair ended at 13:05:36, so no wait on the wall clock.
- Tokens from the Codex session: 1,326,656 in all, 1,223,680 of them cached, 5,766 out.

Billing: the weekly Codex meter moved from 0 % to 1 % during run 2 and stayed at 1 % through run 3, so the runs count against the ChatGPT plan's window and show no API billing.

Three runs are no verdict.

## Built since (tidy, 2026-11-06)

Task `assist-fan-seat-work-alone` (`77e7a9c7`) fixed run 3's conflict: a fanned assistant now gets a fanned seat stencil and directive, whose write surface and `output_files` name the lens reports, and "work alone" there means no other session.
A done fanned seat that returned no lens report is named in the log, and `round-<n>-usage.yaml` records `assist_ending` and `assist_lenses_missing` beside `assist_fan`, so a silent solo seat can no longer pass as a fanned one.
The fix is built but unproved: no real fanned seat has run since, so a real composed fanned seat, with every lens started and returned, is still wanted.

## Built since (tidy, 2026-11-08)

Task `codex-sandbox-mode` (`539525a0`) added the `sandbox_mode` key to `codex.yaml`, `workspace-write` by default and `danger-full-access` as the one other value, set per machine with `lym config codex --layer machine`.
It is the machine-level answer to a Codex seat whose sandbox cannot run shell commands (`bwrap: setting up uid map: Permission denied`); a seat that only could not take the gate lock (run 1) is a different fault, and the key is checked by hand only, not yet on a real Codex run.

## Seen (operator, 2026-11-09)

On OSL-1046 a Codex assist seat read nothing: every command failed with `bwrap: setting up uid map: Permission denied`, and the Chair of `testmap-verb`'s Code-Review waited 19 minutes for the seat's end marker that never came.
The operator says this happens all the time with Codex.

## Plan (operator, 2026-11-09)

No task sets `code_assist_reviewer_model` from now on, so no Codex assist seat runs until Codex works.
The settings on `testmap-verb`, `arras-recipe-perch` and `gate-redundancy-report` were removed.
The trial resumes after Codex's sandbox is fixed on the machine, with the plan of 2026-11-08: the orch sets the setting in the `run_settings` of the next two or three darn tasks the operator releases, with the `review-code` fan, and a darn docs task then reads those rounds.
