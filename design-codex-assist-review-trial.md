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
  It reported an instruction conflict and worked solo; see note `assist-fan-seat-work-alone`.
  So a real fanned seat has still never run.
- Two LOW findings, both real and both fixed in the landing, neither false; the Chair kept both and added a third LOW of its own.
- Time: 212 s from 13:00:44, in parallel; the Chair ended at 13:05:36, so no wait on the wall clock.
- Tokens from the Codex session: 1,326,656 in all, 1,223,680 of them cached, 5,766 out.

Run 4, Code-Review of `assist-fan-seat-work-alone` (`77e7a9c7`), which fixed run 3's conflict, on real code, `astra[medium]` with `review-code` asked for, beside a Chair with its three lenses:

- The seat again ran no lenses and named the same conflict: the run's review used the stencils from before its own fix.
  So the fix's proof, one real fanned seat in which every lens starts and returns, is still to come.
- One BLOCKING and one MEDIUM finding, both on real weaknesses.
  The Chair kept the wiring gap behind the BLOCKING under a finding of its lenses and rejected its rule claim and its fix, and it lowered the MEDIUM to LOW.
- Time: 187 s. Tokens from the Codex session: 1,399,755 in all, 1,295,104 of them cached, 4,969 out.

Billing: the weekly Codex meter moved from 0 % to 1 % during run 2 and to 2 % by run 4, so the runs count against the ChatGPT plan's window and show no API billing.

Four runs are no verdict.
