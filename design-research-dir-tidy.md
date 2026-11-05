# docs/research gets an index and loses its design notes

`research-dir-tidy` · Task · enhancement · draft

Depends on: [`readme-refresh`](design-readme-refresh.md)

## Seen (readme-refresh sampler, 2026-11-05)

`docs/research` is hard to find one's way in:

1. It mixes flat `.md` files, dated directories with a `report.md`, and directories with a `README.md` and scripts.
2. Long near-duplicate series: `codex-astra-*`, `codex-v2-p*`, `glyph-audit-*`, `creel-audit-*`, `twill-*`.
3. Some entries are design notes, not research: `reed-proposal.md`, `reed-exploration.md`, `logging-inventory.md`, `quarry-holistic-fix-log.md`, `weaving-glossary.md`.
4. No index.

## Cost of moving studies

- About 30 studies' scripts find the repository root by `../../..`.
- `internal/lymcwd/docslink_test.go`'s `docsLinkIsResearchData` requires `docs/research/<report>/data/`.
- Code comments cite exact paths, such as `codex-spike-2026-10-24`, `session-fork-spike.md`, `batch-budget-trial-2026-11-01` and `reed-hooks-exploration.md`.
- Reports' relative links break on a move, and `README.md` now links several studies under `## Research on its own runs`.

## Chosen (operator, 2026-11-05)

- A topic-grouped index `docs/research/README.md` lists every study once, with a line on what it measured; a near-duplicate series is one index line pointing at its newest member.
- The design notes move out of `docs/research` to `docs/` (or beside the design they serve), with every link and code comment naming them updated.
- Every study stays where it is: no study moves, is renamed or is deleted.
