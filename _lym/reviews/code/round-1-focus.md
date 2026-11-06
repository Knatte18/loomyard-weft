---
round: 1
focus:
  - "docs/research/codex/review/assist-fan-trial-2026-11-06/report.md: does it report, for each lens, whether a subagent started, whether it returned, and whether the assistant waited for all inside its turn, and does the evidence in data/ bear each claim out?"
  - "report.md: does the claim that the report parses as a review name the check `ReviewGateEntry` names, and does data/review.md satisfy what that check requires?"
  - "report.md: are wall time and tokens given with their source, and is any figure stated that Codex's records or the data/ files do not support?"
  - "report.md: does the comparison against `round-1-review.md` and the solo report of weft commit `327f541d1d` state that the earlier review read pre-fix code, and does each finding listed as matched or missed hold against data/review.md and the lens files?"
  - "Scope: the diff should add only the report and its data files; check that no source, stencil or contract file changed, and that the report names part 2 as still to come without claiming a result for it."
  - "data/prompt.md: is it built from the stencils and lens set the decision record names (`burler-template-assist-reviewer`, `burler-assist-cluster-fan`, Codex `assist-reading` and `assist-fan`, the `review-code` lenses in `lenses.yaml`), and does the report's description of the run match it, including that it reviews commit `b93a0a52`?"
  - "Bounds: does the report show at most one run and one retry, with any retry caused by a launch fault unrelated to the fan?"
  - "If the report says the fan failed or partly failed, does it state what failed and where, and record the fix as a note rather than a code change?"
---

Single-commit diff adding a research report and its captured run data.
No code changes, so the focus is on whether the report's claims match the captured evidence and the bounds in the decision record.
