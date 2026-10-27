## Goal

Run one forked Codex Astra fan on the `P4` plan review with the context parity of the sibling bench plus one tuned-prompt block, and report whether the block makes the fan find the Claude fan's three `BLOCKING` findings.

## Scope

- In: `docs/research/codex-astra-tuned-prompt-bench-2026-10-27/` with `report.md` and `data/`, landed as one commit; the harness is copied from the forked fan bench's `data/harness/` and edited only for paths and the context.
- In: one `codex exec` handler on `gpt-6-astra` at high effort spawning 3 lens forks with `fork_turns="all"`; no repeat, no Claude run.
- In: grading of the handler's findings and of any dropped fork finding against the `P4` fan reference, in `data/match.tsv`.
- Out: any change to Go code, stencils, `PATTERN.md` or `docs/` outside the new report directory.

## Decisions

### Context matches the sibling bench

Decision: the handler prompt inlines the PATTERN directive as production renders it for the burler reviewer, and the snapshot root holds the snapshot's `CLAUDE.md` as `AGENTS.md`.
The directive is the code commit's `pattern-directive-review-fix.md` stencil filled with that commit's `PATTERN.md` and an empty `pattern_changes`; it is the text `pattern.Directive` returns for `RoleReviewFix`.
Rationale: the sibling task `codex-astra-context-parity-bench` runs the same context, so the two runs differ only in the block.
The sibling had written no files when this record was written, so the context files are built from the board entries' shared specification; the report records their sha256 so a reader can compare them with the sibling's.

### One tuned block, written once

Decision: the block lives in `data/prompts/tuned-block.md` and is appended to the handler prompt and, through the handler's fork boilerplate, to each fork's prompt.
It asks for three kinds of reading, the claim check on existing code and callers, the `pattern/` background of each touched PATTERN entry with a placement check of each new command, package or file, and the added tests' waits and sleeps; it names no defect of the reference.
Rationale: the task fixes the block's three clauses and forbids naming reference defects.

### Forks receive the block through the boilerplate

Decision: the cluster rules forbid adding direction to a fork's prompt, so the block is placed in the handler prompt before the instruction files and in the boilerplate's text, and the fork inherits the handler's context with `fork_turns="all"` as well.
Bound: the edit adds the block to the boilerplate and nothing else; the tool-call cap of 6 stays, so a fork's extra reading is bounded by it.
Rationale: the task asks for the block in each fork's prompt, and a fork with an inherited context holds the block either way.

### Stop conditions

Decision: stop and report if a rate-limit window passes 70 percent, if the rollouts show no concurrent forks inheriting context, or if `AGENTS.md` was not loaded.
Rationale: the task's cap and its method's confirmation step.

### Grading

Decision: one session grades as the earlier benches did, `shared`, `unique-real` or `false` against the `P4` fan reference, and the report names for each of the three `BLOCKING` findings whether this run found it.
Rationale: the earlier benches' rubric keeps the rows comparable.

## Constraints

- `gpt-6-astra` at `high` effort only, never `xhigh`.
- One run, no repeat; no Claude agent run for the bench.
- Scratch files live under `.scratch/` and are not committed; `.scratch/codex-home` holds only a link to the operator's `auth.json`.
- Markdown uses semantic line breaks.

## Auto-mode assumptions

- The sibling's context files are not available, so this run builds them from the shared specification.
- The `P4` snapshot is rebuilt from the forked fan bench's `data/inputs.txt` commits, present in this repository and in `loomyard-weft`.

## Open risks

- The tuned block and the forks' 6-call cap interact: a fork may not be able to read the extra files the block asks for.
- One input and one run cannot separate the block's effect from run-to-run variance.

## Acceptance criteria

- `report.md` states the result per `BLOCKING` finding, the summary row beside the forked fan bench's and the Claude fan's rows, the per-call tokens and wall times, and the windows.
- The rollouts show three concurrent forks that inherited the handler's context and a loaded `AGENTS.md`, or the report states the stop.
- `data/match.tsv` grades every handler finding and every dropped fork finding.
- `data/` holds the harness, the prompts including the tuned block, the context files' hashes, the reviews, the usage and the rollout evidence.
