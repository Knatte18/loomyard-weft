# Decision record — codex-review-bench

## Goal

Measure whether Codex reaches the Claude reviewers' quality as a Burler reviewer of discussions and plans, and at which model and effort, before note `codex-engine` is built.
The deliverable is one report, `docs/research/codex-review-bench-2026-10-25/report.md`, with a summary table and a conclusion.
Codex spends its own tokens; Claude drives the bench and grades the findings.

## Scope

In:

- A fixed set of finished loom runs from the history of `loomyard-weft`, each with its round-1 discussion or plan review by Claude.
- Headless `codex exec` runs of the review over that set, over the models `gpt-6-luna`, `gpt-6.1-sol` and `gpt-6-astra` and a range of efforts.
- Per input and setting: findings Claude also found, real findings Claude missed, false findings, Claude findings Codex missed, tokens and wall time.
- A test of `codex exec` itself and of billing under the ChatGPT login: plan rate-limit use per run.
- The report, and the raw review files and score tables it rests on, beside it.

Out:

- Production code, stencils, config and PATTERN: nothing under `internal/`, `cmd/`, `contracts/` or `PATTERN.md` changes.
- A `codexengine`, a Codex shuttle engine or any wiring of Codex into a loom run (note `codex-engine`).
- Code-review inputs: the bench covers discussion and plan reviews only.
- Any change to `~/.codex`.

## Decisions

### D1 — Inputs are round-1 reviews of landed loom runs

Take round-1 discussion reviews and round-1 plan reviews from weft commits where the review file was written, each paired with the artifact the Claude round saw: the decision record and support log, or the plan, from the same weft tree.
Round 1 only: a later round reviews a record the fixer already changed, and the Claude review of it names an amendment, not the artifact.
Choose the runs by the Claude report: varied findings and verdicts, and at least some with `BLOCKING` findings, so the set holds both defects to find and artifacts to approve.
The set is a first sample of discussions and a smaller sample of plans, sized by D6's budget; the report states the exact set.

Rationale: round 1 is the only round whose input is recoverable from one weft tree without replaying fixer edits.

### D2 — Code is read at the commit the Claude round saw

Each input's code snapshot is `git archive` of the `loomyard` repository at the last commit of the run's parent branch before the Claude review's recorded `start`, extracted under `.scratch/`; no worktree is created.
The snapshot's commit is recorded per input.

Rationale: the review's prompts read the repository, so findings are comparable only against the same code; the parent's HEAD at round start is the nearest recoverable commit.

### D3 — The prompt is the burler reviewer chain, solo, over the Claude round's own rubric and focus

The prompt joins `burler-template-reviewer.md`, `burler-step-1-explore.md`, `burler-step-review.md`, `burler-review-format.md`, the Discussion or Plan rubric with amendment `none`, the round's focus directive when the Claude round had one, and the solo cluster rule.
Paths in it point at the scratch snapshot and the artifact copies; Codex writes the review file in the format the stencil pins.
Codex reviews solo; the Claude round used a lens fan, so lens-only findings are Claude findings a solo reviewer has no peer to cover, and the report says so when comparing.

Rationale: the stencils are the production prompt, so what is measured is what a Codex Burler reviewer would be given.

### D4 — Codex runs isolated, sandboxed to its scratch directory

Each run is `codex exec` with `CODEX_HOME` set to a directory under `.scratch/` that links `~/.codex/auth.json`, a working directory under `.scratch/`, `-s workspace-write` and approvals off.
The operator's `~/.codex` is read through the auth link only and never written.
What this can let through: Codex can read and write only under its working directory, with network off, so it cannot touch the worktree's tracked files, the weft, or the operator's Codex state.

Rationale: the board entry requires the operator's Codex home untouched.

### D5 — Grading is by Claude against the code, with the run's own outcomes as evidence

Every Codex finding is matched to the Claude findings of the same round and labelled `shared`, `codex-only real`, or `false`; every unmatched Claude finding is `claude-only`.
`codex-only` findings are judged by reading the cited artifact and code at the D2 commit.
The round's fixer report and bouncer verdict are read as evidence of what the pipeline accepted as real.
The match table is committed beside the report, so the grading can be re-read.

Rationale: no ground truth exists beyond the artifact and the code; a table the reader can audit bounds the grader's bias.

### D6 — The bench is metered and stops on rate limits

After each run the bench reads the rollout's `token_count` rate-limit fields and records fresh and cached input, output and reasoning tokens, wall time and the percent of each plan window used.
It stops, and the report says so with the point reached, when either window shows 70 percent used.
The effort sweep runs only on a subset of inputs, after the base grid of models at one effort is done.

Rationale: the plan is a ChatGPT Plus login with 5-hour and weekly windows; the entry requires an early stop before they run out.

### D7 — The report lands under `docs/research/` with the data beside it

`docs/research/codex-review-bench-2026-10-25/report.md` holds the summary table, the method, the `codex exec` and billing findings, the limits of the comparison and the conclusion.
The Codex review files, the per-run usage table and the match table go in `data/` beside it.
The report is a measurement record: a later run supersedes it with a new dated directory.

Rationale: `docs/research/` is where the earlier Codex spikes live.

## Constraints

- No file outside `docs/research/codex-review-bench-2026-10-25/` is committed, and the commits touch only text no build or test reads.
- Scratch lives under `.scratch/` and is never committed.
- `~/.codex` is never edited; Codex's own write of a trust entry goes to the scratch `CODEX_HOME`.
- Codex's work directory is never the worktree root.
- `lyx loom fmt` does not apply; no Go file changes.
- Markdown follows semantic line breaks; table cells stay on one line.
- A run's agents never create a task: any follow-up goes on the board as a note.

## Auto-mode assumptions

- The operator's list "Luna, Sol, Astra" means `gpt-6-luna`, `gpt-6.1-sol` and `gpt-6-astra` from the local model cache; `gpt-6-sol` and `gpt-5.6-*` are not benched.
- Round-1 reviews are representative of later rounds for the purpose of ranking models.
- The parent branch's HEAD at the Claude review's start stands in for the exact commit the reviewers read.

## Open risks

- Rate limits may cut the grid short; the report then states the unmeasured cells.
- A solo Codex review set against a lens-fan Claude review may undercount Codex; D3 states the caveat.
- Grading by one Claude session is a single judge; D5's committed table is the audit trail.

<!-- lyx:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/code/round-1-review.md; fixer report: _lyx/reviews/code/round-1-fixer-report.md.
  - blocking-counts-not-recomputable (class design, severity MEDIUM, open in rounds 1)
  - config-toml-byte-identical-unevidenced (class design, severity MEDIUM, open in rounds 1)
  - d5-grading-unchecked-rows (class scope, severity MEDIUM, open in rounds 1)
  - grid-no-70-percent-stop (class scope, severity MEDIUM, open in rounds 1)
  - grid-skips-refused-cells (class design, severity MEDIUM, open in rounds 1)
  - usage-txt-missing-window-percentages (class design, severity MEDIUM, open in rounds 1)
  - workspace-write-reads-unconfined (class design, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Code-Review -->

## Acceptance criteria

- The report exists at the D7 path with a summary table of model and effort against shared, codex-only real, false and claude-only counts, tokens and wall time.
- The report states a conclusion on whether Codex reaches Claude's review quality and at which model and effort, or says the set cannot support one.
- The report records how `codex exec` behaves headless and what each run cost against the plan's rate limits under the ChatGPT login.
- The report names the exact inputs and each input's code commit.
- Every Codex run uses a scratch `CODEX_HOME`, and `~/.codex/config.toml` is byte-identical before and after.
- A stop on rate limits is stated in the report with the cells it left unmeasured.
- No production file changes: the diff is under `docs/research/codex-review-bench-2026-10-25/` only.
