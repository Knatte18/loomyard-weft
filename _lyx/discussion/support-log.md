# Support log — codex-review-bench

## Interview

A darn run holds no interview.
The writer read the board entry, the board (no entry contradicts it; `codex-engine` is the downstream note, `umbrella-daemon` is the only run in flight and touches none of this), `lyx loom peers`, the loom discussion spec and the final-summary spec.
It read the two earlier Codex reports under `docs/research/`, the Discussion-Review rubric, the burler reviewer stencils, and the weft history of `_lyx/reviews/discussion` and `_lyx/discussion`, which holds the record, the support log and the round-1 review at the weft commit where the review was written.
It confirmed that `codex` 0.164.0 is installed, `~/.codex/auth.json` exists, and the model cache lists the three models.

## Rejected alternatives

- **Interactive Codex in tmux.** The spike showed Esc and paste quirks; `codex exec` is the headless route the entry names and costs no pane handling.
- **Reviewing later rounds.** The input is an already-fixed record and the Claude review names an amendment; round 1 is cleaner.
- **A git worktree per code commit.** Worktree creation is not this run's to do; `git archive` into `.scratch/` gives the same files.
- **Auto-grading by a Claude judge session per finding.** A second model run adds cost and a second bias; one grader with a committed table is enough for a first bench.
- **Lens-fan Codex reviews.** The fan multiplies Codex tokens per run and would hit the plan limits first; solo is the unit a Codex Burler seat would start from.
- **Benching `gpt-6-sol` and `gpt-5.6-*`.** The entry names three models; older slugs only widen the grid.

## Review rounds

_No rounds yet._

## Question ledger

- Which models does "Luna, Sol, Astra" name? Resolved by auto-pick: `gpt-6-luna`, `gpt-6.1-sol`, `gpt-6-astra`, the newest of each in the local model cache.
- How many inputs? Resolved by auto-pick: a first sample sized by the rate-limit budget, stated in the report; the stop rule is D6.
- Which commit did the Claude round see? Resolved by auto-pick: the parent branch's HEAD before the round's recorded `start`.
- Is the entry outdated? Resolved: no; the board holds no contradicting entry and no `Decision for codex-review-bench:` note.
- Do the plan reviews carry a recoverable artifact? Open until the first plan input is extracted; if a plan tree cannot be recovered, that input drops and the report says so.
