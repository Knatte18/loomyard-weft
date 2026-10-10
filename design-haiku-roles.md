# Haiku 5.5 for the light roles: driver and judge

`haiku-roles` · Note · enhancement, config, loom, burler · 

## Seen

The operator benchmarked Haiku 5.5 on 2026-10-08 (`docs/benchmarks/haiku-5-5.md`): on one fully specified coding task it scored as well as Sonnet and Opus at medium, at a fraction of the cost, but took about twice as long and grew its context to 74–132k.
Today `_lyx/config/loom.yaml` runs the loom driver and every Bouncer row's seed and judge passes on `sonnet[medium]`.
The driver steps one run through `lyx shed step` and answers its waits, and the judge rules on a fixed question per round; neither writes code or long prose.

## Idea

Run the driver and the judge on Haiku 5.5 where it holds (operator, 2026-10-08: most likely only these two; the other roles stay as they are).

- Set `driver` and `judge` in `loom.yaml` to a Haiku model-spec on one trial run, and compare against a Sonnet run from the usage records: interventions by the hub, judge verdicts later overturned, and token cost.
- Keep the change only for a role whose trial shows no more interventions or overturned verdicts.

## Driver switched (operator, 2026-10-08)

`driver` is `haiku[medium]` in `loom.yaml` from 2026-10-08, for runs started after that; the judge stays on Sonnet until the driver trial is read.
Compare the driver row of the token report per role, and the hub interventions, against the Sonnet runs before it.

## Seen in use



- 2026-10-10, `bolt-and-store`: the Haiku 5.5 driver read its prompt, summarized its rules, and ended the turn asking "Tell me when to start", although the prompt says it runs autonomously with no operator to ask.

  The Haiku drivers of `multi-llm-producer` and `daemon-wakeups`, on the same prompt, started stepping at once; a message from the orch got it going.

  The driver stencil (`contracts/stencils/shed/shed-template-driver.md`) never says "start now"; a line that reading the prompt is the go-ahead and the first turn launches the first step would close it.
  Fixed in 189fd59c8: the driver stencil now says the prompt is the go-ahead and a turn never ends on a question.
