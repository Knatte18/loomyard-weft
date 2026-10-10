# Haiku 5.5 for the Bouncer judge: is its ruling simple enough

`haiku-roles` · Note · enhancement, config, loom, burler · 

## Seen

The operator benchmarked Haiku 5.5 on 2026-10-08 (`docs/benchmarks/haiku-5-5.md`): on one fully specified coding task it scored as well as Sonnet and Opus at medium, at a fraction of the cost, but took about twice as long and grew its context to 74–132k.
The driver steps one run through `lyx shed step` and answers its waits, and the judge rules on a fixed question per round; neither writes code or long prose.

## Driver switched (operator, 2026-10-08)

`driver` is `haiku[medium]` in `loom.yaml` from 2026-10-08, for runs started after that.
Compare the driver row of the token report per role, and the hub interventions, against the Sonnet runs before it.

## What the task decides: the judge (operator, 2026-10-10)

Switching the judge is one config key; the task's work is deciding whether it is sound.
The question is whether the judge's job is simple enough for Haiku 5.5: read what each Bouncer row's judge and seed passes receive and must rule on, and judge where a wrong verdict lets a defect through or sends good work back.

- Read the judge stencils and a sample of past verdicts from the usage and review records, and name the rows, if any, whose ruling needs more than Haiku holds.
- Trial the judge on Haiku on one run and compare against Sonnet runs: verdicts later overturned, extra review rounds, hub interventions and token cost.
- Switch only the rows where both the reading and the trial hold; the other roles stay as they are.

## Seen in use

- 2026-10-10, `bolt-and-store`: the Haiku 5.5 driver read its prompt, summarized its rules, and ended the turn asking "Tell me when to start", although the prompt says it runs autonomously with no operator to ask.
  The Haiku drivers of `multi-llm-producer` and `daemon-wakeups`, on the same prompt, started stepping at once; a message from the orch got it going.
  Fixed in 189fd59c8: the driver stencil now says the prompt is the go-ahead and a turn never ends on a question.

## Analysis and decision (operator, 2026-10-10)

The analysis is in `docs/research/haiku-judge.md` (5459376bb): keep Sonnet on the Discussion-Bouncer judge; trial Haiku 5.5 on the Plan-Bouncer and Webster-Bouncer judges; seed passes are safe for Haiku.
The `judge` key sets all three rows at once, so the judge stays on Sonnet for now.
The trial waits for the note `per-role-models`, which gives each row's judge its own model key, and then runs Haiku 5.5 on the Plan-Bouncer and Webster-Bouncer judges only.
