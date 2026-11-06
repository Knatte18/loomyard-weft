# A fanned assistant seat is told to work alone, and ran without its lenses

`assist-fan-seat-work-alone` · Task · bug, burler, stencil, codex · running · Drift-Baseline

## Seen

The first real fanned assistant seat, Code-Review of `codex-assist-fan-trial` (`2c4864bb`, `code_assist_reviewer_fan: review-code`, `astra[medium]`), ran no lenses.
Its report says so under "Instruction conflict": "The top-level task says to work alone and permits writing only this report; instruction 2 asks for subagents and additional lens files. I followed the top-level restriction".
Its Codex session holds no `spawn_agent` call and no subagent rollout.
`round-1-usage.yaml` still records `assist_fan: review-code`, and nothing in the round flagged the missing fan: the Chair put the note under Rejected as no defect of the diff.

## Cause

The composed prompt contradicts itself:

- `contracts/stencils/seat/seat-directive-assistant.md` says "You work alone: you write your output and nothing else", and "Your output is the only file set you write", with `output_files` naming the report alone.
- `contracts/stencils/burler/burler-template-assist-reviewer.md:20` says the write surface is the report "and nothing else".
- `contracts/stencils/burler/burler-assist-cluster-fan.md` asks for subagents and lens reports, and calls those reports "the one exception".

The hand-composed prompt of the same trial's part 1 kept the "work alone" block, but its write-surface line named the lens reports, and Astra fanned there.
That is one run each, so this difference is the likely cause and not a proven one.

## Idea

- When the seat is fanned, the seat directive and the write surface say so: "work alone" means no other session, not no subagents, and `output_files` and the write-surface line name the lens reports.
- A round whose assistant was asked to fan and returned no lens report says so in its record, so a silent solo seat cannot pass as a fanned one.

Proof wanted: one real fanned seat in which every lens starts and returns.
