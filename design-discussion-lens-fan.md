# Discussion writer forks lens forks after exploring, before writing

`discussion-lens-fan` · Note · enhancement, loom, stencil

## Seen (2026-10-11)

The first Discussion bench (`docs/research/discussion-bench.md`, 7d439cf71) found that no seats chair with an advisor wrote a better draft than the same chair alone, and that an Opus or high-effort Sonnet advisor roughly doubled the cost.
Each advisor is a standalone session at full price that re-reads the code itself; the operator names this as part of the problem.
Each configuration ran once on one task, so the noise is unmeasured.

## Idea (operator, 2026-10-11)

Bring Burler's lens fan into Discussion, inside the writer's own session:

1. The writer, on a strong model such as `opus[medium]`, explores the task and code in full itself.
2. After exploring and before writing, it forks itself once per lens, the way Burler's fanned reviewer forks its lens forks.
3. Each lens fork returns a short list of what the decision record must settle in its area: risks, cases to handle and open questions; it proposes no decisions.
4. The writer writes the decision record with every list in front of it.

Gains:

- Every fork runs on the writer's strong model.
- Every fork inherits the writer's context, so it reads the shared prefix from cache rather than re-reading the code.
- No seats engine or SendMessage rounds: a writer stencil change and lens texts, which Burler's fan already has.

Choices made with the operator:

- Fork before the draft, not after: forks spawned after the draft inherit it and tend to polish its decisions rather than question them.
- Lists, not decisions: lists merge by union, while competing decisions from several lenses would need resolving.
- Left in place: every fork has read the code the way the writer did, the price of the cache hits; only standalone sessions read it independently.
- Burler's Discussion-Review still reviews the draft after.

## Bench

Compare, three runs each on the same reference task, so the spread is measured too:

- `opus[medium]` writer with the lens fan;
- `opus[medium]` alone;
- `opus[high]` alone, the first bench's best draft.

Measure wall time, price per configuration, the draft against the reference final discussion judged on the merits, and where possible how many Discussion-Review rounds each draft needs.
The open question whether Burler's fan findings are worth their cost applies here too.
