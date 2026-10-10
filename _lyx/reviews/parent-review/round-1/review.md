# Parent review — multi-llm-producer, round 1

```yaml
verdict: approve
reviewer: hub orchestrator (parent-review fork)
```

The decision record does what the board entry asks, no more and no less.
`MultiLLMProducer` sits in `internal/shedadapters` beside `SingleLLMProducer`.
It runs N seats in one Shed step with one outcome.
The settled terms are kept: seat, `chair`, `advisor-1`…, and no "master".
A message to the producer goes to the chair.
`creel` is not used.
No recipe row, Burler or Discussion uses the producer, and it is proven by tests and a fixture recipe.
Every open question on the board is settled: D6 (advisor-to-advisor messaging), D7 (failure and hang), D5 and D7 (how the producer knows an advisor is done), D9 (layout) and D11 (`SingleLLMProducer` and the N = 1 case).
The two new seams beyond the producer serve the board's own one-table requirement and the advisors' quiet turns, and are bounded in the record:

- the stencil include form (D3);
- the shuttle quiet-hold knob (D10).

No finding.

Operator check before submit: D6 reads the board's "own internal message channel" as Claude Code's `SendMessage` by strand name (assumption A1), not as a channel the producer carries itself.
