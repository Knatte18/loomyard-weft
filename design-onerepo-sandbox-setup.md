# One-repository sandbox: create the repo and clone the hub

`onerepo-sandbox-setup` · Task · research, fabric · running · Drift-Recheck

## Goal

First of three trials of the one-repository hub layout (57015a52) in a new sandbox, driven from this task's worktree; the run acts as the sandbox's operator and orch.

## Steps, exact cap

- Create one private GitHub repository `Knatte18/lym-sandbox-onerepo` with `gh repo create --private` (operator approved, 2026-11-21), seeded with a tiny Go module (one package, one test) on `main`.
- Clone it once as a one-repository hub with `lym fabric clone --single-repo` under `/home/knatte/Code/lym-sandbox-onerepo-LYMHUB`.
- Check and record: `gh api` branch and tag lists show only the code branch; `git ls-remote` shows the board and prime records branch under `refs/lym/heads/`; `lym board list` and `lym fabric` status work in the hub.
- No other repository is created; nothing is deleted.

## Output

A report under `docs/research/single-repo/sandbox-1-setup-2026-11-21/report.md`: each command, its outcome and every friction or failure.
Each lym bug found is filed as a note on this hub's board (kind note, labels from `lym board labels`).
