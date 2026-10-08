# Haiku 5.5 for the light roles: driver and judge

`haiku-roles` · Note · enhancement, config, loom, burler

## Seen

The operator benchmarked Haiku 5.5 on 2026-10-08 (`.scratch/haikubench` in the prime, not in git).
Today `_lyx/config/loom.yaml` runs the loom driver and every Bouncer row's seed and judge passes on `sonnet[medium]`.
The driver steps one run through `lyx shed step` and answers its waits, and the judge rules on a fixed question per round; neither writes code or long prose.

## Idea

Run the driver and the judge on Haiku 5.5 where it holds (operator, 2026-10-08: most likely only these two; the other roles stay as they are).

- Set `driver` and `judge` in `loom.yaml` to a Haiku model-spec on one trial run, and compare against a Sonnet run from the usage records: interventions by the hub, judge verdicts later overturned, and token cost.
- Keep the change only for a role whose trial shows no more interventions or overturned verdicts.
