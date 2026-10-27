## Goal

Measure whether Codex Astra's forked lens fan at high effort finds more of the Claude fan's findings on plan `P4` once it starts from the context the Claude reviewers had: the PATTERN overview inlined in the prompt as production renders it, and the snapshot's `CLAUDE.md` loaded as `AGENTS.md`.
The result is a report under `docs/research/codex-astra-context-parity-bench-2026-10-27/` with its `data/`.

## Scope

- One input: `P4`, the `machine-handover` plan, rebuilt from the forked fan bench's `data/inputs.txt` (weft commit and code commit).
- One run: one `codex exec` handler at `gpt-6-astra`, high effort, spawning three lens forks with `fork_turns="all"`; no repeat, no Claude run.
- Change only the context against the forked fan bench: a `pattern_directive` block in the explore step, and `AGENTS.md` in the snapshot root.
- Harness scripts are copied from the forked fan bench's `data/harness/` with their paths moved to this worktree, plus one script that stages the context.
- Output is documentation and data only; no Go code, PATTERN entry or module doc changes.

## Decisions

- **The PATTERN block is the production `RoleReviewFix` directive.**
  The staging script fills the code commit's `pattern-directive-review-fix` stencil, banner stripped, with the snapshot's `PATTERN.md` and the declared-changes fragment built from the decision record's `## PATTERN changes` bullets, and puts the result in the explore step's `pattern_directive` marker.
  Rationale: `burlerengine.Review` renders exactly that for a reviewer, so the handler sees what the Claude round-1 reviewer saw.
  Bound: forks inherit it through `fork_turns="all"`; no lens prompt is edited.
- **`AGENTS.md` is a copy of the snapshot's `CLAUDE.md`, placed in the snapshot's root, the handler's `-C` directory.**
  Rationale: Codex loads `AGENTS.md` as a Claude session loads `CLAUDE.md`.
  Whether it was loaded is read from the handler's rollout; a run where it was not is reported, not graded.
- **The remaining context differences are recorded in the report, not closed.**
  Plugin skills, the user-level `CLAUDE.md`, Claude's system prompt and the Claude session's tool set have no Codex counterpart here.
  Rationale: the task asks for the differences to be listed, not removed.
- **The stop conditions are the task's.**
  The run is abandoned and reported when the forks did not start, did not run concurrently, did not inherit the handler's context, or `AGENTS.md` was not loaded in the rollout, and when a window passes 70 percent.
  Rationale: a run failing those checks measures a different thing than the forked fan bench's.
- **Grading follows the earlier benches.**
  Each handler finding, and each fork finding the handler dropped, is `shared`, `unique-real` or `false` against the fan reference of `P4`, in `data/match.tsv`, and the report says for each of the Claude fan's three `BLOCKING` findings whether this run found it.
  Rationale: the figures must sit beside the forked fan bench's row and the Claude fan row.

## Constraints

- `gpt-6-astra` at high effort for the handler and its forks; never `xhigh` or above.
- One fork-fan run; the scratch `CODEX_HOME` links the operator's `auth.json`, and the operator's `config.toml` does not apply.
- Scratch work lives under `.scratch/`, uncommitted; the committed harness holds scripts only.
- Markdown follows semantic line breaks; table cells stay on one line.
- The report states measured figures as one run's record and claims no ranking from one input.

## Auto-mode assumptions

- The date in the output directory name is 2026-10-27, the day of the run.
- The fan reference of `P4` is the forked fan bench's list of 11 findings, with the same ids, taken from its report and `match.tsv`.
- The grader is the writing Claude session, as in the earlier benches.

## Open risks

- Codex may not load `AGENTS.md` from a `-C` directory outside a git repository; the rollout decides, and a failed load is reported with the run stopped.
- The run shares the weekly window with a concurrently running sibling bench; the windows are read from the rollouts and the 70 percent stop applies to both.
- One input and one run per cell: a difference of one or two findings is inside the noise.

## Acceptance criteria

- `docs/research/codex-astra-context-parity-bench-2026-10-27/report.md` exists with `data/` holding the harness, prompts, `inputs.txt`, `usage.txt`, `rollout-evidence.txt`, `match.tsv` and the reviews.
- The report records the wall time, the handler's and each fork's fresh input and output tokens, and the windows.
- The report confirms from the rollout that the forks started, ran concurrently and inherited the handler's context, and that `AGENTS.md` was loaded.
- The report lists every remaining difference from the Claude round's context.
- The report has a summary table with this run beside the forked fan bench's row and the Claude fan row, and says per `BLOCKING` finding of the Claude fan whether this run found it.
- Every handler finding, and every dropped fork finding, is graded in `data/match.tsv`.
