---
round: 2
focus:
  - "In docs/research/codex-review-bench-2026-10-25/, check whether data/usage.txt carries per-run five-hour and weekly window percentages and whether the percentages quoted in report.md's Billing section (the 68 and 79 percent readings, the per-run points) follow from it."
  - "In report.md under the Codex runs section, check whether the claim that ~/.codex/config.toml is untouched is stated only as far as its evidence goes, and whether the auth.json symlink and the ~/.codex directory mtime change are explained."
  - "Check whether the BLOCKING totals and the Claude reference tokens and wall times quoted in report.md can be recomputed from files committed under data/ (match.tsv, inputs.txt and any sibling file)."
  - "In data/harness/run.sh, which runs codex with -s workspace-write, check whether report.md states that reads are not confined to the work directory and shows or discloses whether any run read a path outside it."
  - "In data/harness/grid.sh, check whether a cell whose run was refused (result.txt written with a nonzero rc) is run again on a rerun, and whether the window readout can pick a rollout from an earlier run."
---

The focus lists the places where round 1 found gating defects, and each entry asks a question to settle.
