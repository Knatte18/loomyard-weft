---
round: 1
focus:
  - "Diff paths: does every changed path sit under `docs/research/codex-review-bench-2026-10-25/`, and is any scratch material (`.scratch/` contents, snapshots, `CODEX_HOME`, auth links or credentials) committed?"
  - "`data/harness/*.sh`: do the scripts set a scratch `CODEX_HOME` and a work directory that is never the worktree root, link `~/.codex/auth.json` read-only, and use `-s workspace-write` with approvals off as D4 says? Can any path write to `~/.codex`?"
  - "`report.md` summary table against `data/match.tsv` and `data/usage.txt`: do the shared, codex-only real, false and claude-only counts, tokens and wall times in the table follow from the committed data? Check a few cells by recounting."
  - "Grid coverage: the report must name every unmeasured cell (missing model runs such as `gpt-6-astra` on D6 and the P inputs, and any effort sweep not run) and give the reason. Do the review files under `data/reviews/` match what the report claims ran?"
  - "D6 rate-limit stop: does `data/usage.txt` carry the per-run window percentages the report cites, and does the report state whether and where the 70 percent stop fired?"
  - "Inputs and commits: does the report name the exact inputs and each input's code commit (D1, D2), consistent with `data/inputs.txt`, and does it say that only round 1 was used?"
  - "Conclusion against the acceptance criteria: does it say whether Codex reaches Claude's review quality and at which model and effort, or that the set cannot support one, and does the evidence supports the strength of the claim given the sample size, the solo-versus-lens-fan caveat (D3) and the single judge (D5)?"
  - "`codex exec` and billing section: does the report record headless behavior and plan rate-limit use per run under the ChatGPT login, and is the `~/.codex/config.toml` byte-identical claim backed by something in the data?"
  - "Repository rules: semantic line breaks in the `.md` files, table cells on one line, and no perishable tallies pinned in prose outside the measurement report."
---

Round 1 seed. The diff is a research report with its data; no Go or production file is expected to change.
Mechanical gates do not cover the numbers in the report, so the entries lean on checking them against the committed data.
