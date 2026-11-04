# Measure how Plan and Discussion writers explore and whether they use lookup subagents

`planner-fork-lookups` · Task · research, loom · running · Drift-Verify

## Seen (orch, 2026-11-01)

In a sample of 28 sessions from the batching study, the single largest cost was the Plan writer of `run-verbs-shared` in `plan-split-bench`: 31 chained grep turns at 390k to 530k context and 15 one-file-per-turn fix turns at 630k to 668k.
Every lookup the writer makes is paid at its whole context, so exploration late in a large session is the expensive part of planning.
Discussion-Write explores the code more than any other role, so the same cost applies there.

Since task `explore-subagent-model` landed (2026-11-02), the Plan-Write, Discussion-Write and darn writer stencils encourage the writer to hand batches of independent lookups to fresh subagents on the `subagent_model` of `loom.yaml` (default `sonnet`), which return only what is relevant with paths and line ranges.
The guidance encourages; it does not command.
`turn-savings-measure` (`docs/research/turn-savings-measure-2026-11-04/report.md`) measured darn and Webster only, not these two writers.

## Measurement (operator, 2026-11-04)

A measurement from transcripts of landed runs only; no agent run, no replay and no bench.

- Sample: the Plan-Write and Discussion-Write sessions of the 10 most recent tapestry runs that landed after 2026-11-02, both roles of each run.
- For each writer session, count:
  - its exploration turns (turns made only of lookups: Read, Grep, Glob, or a read-only Bash lookup such as `grep`, `rg`, `find`, `ls`, `cat`, `sed -n` or a `git` read), with the context and cost at each;
  - chains of exploration turns in a row, their length and the context they ran at;
  - the subagents it started for lookups, how many lookups each batch held, and the subagent's model and cost;
  - how much of its exploration cost fell above 200k context.
- Report whether the writers follow the stencil guidance, and how much of their exploration cost a writer could have moved to subagents up front, with the chains that show it.
- Reuse the method and scripts of `docs/research/turn-savings-measure-2026-11-04/` where they fit.
- The report closes with a recommendation: whether a bench of the guidance (writer with it against writer without it, on one `plan-split-bench` input) is worth running, or whether the guidance should become stronger, or nothing.
- The report lands under `docs/research/`.
