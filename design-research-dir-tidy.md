# docs/research is restructured into topic folders with an index

`research-dir-tidy` · Task · enhancement

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

`docs/research` is restructured into topic folders, not only indexed.

- Every study moves into a topic folder, such as `codex/`, `twill/`, `glyph/`, `creel/`, and one per other theme the studies fall into; a near-duplicate series becomes one folder of its own inside its topic.
- One layout per study: a study with data or scripts is a directory with a `report.md`; a lone report may stay a single file.
- The design notes move out of `docs/research` to `docs/` (or beside the design they serve).
- A topic-grouped index `docs/research/README.md` lists every study with a line on what it measured.
- Nothing is deleted.

Every reference is fixed in the same change, so nothing goes stale:

- the scripts that find the repository root by `../../..` find it by a depth-independent means, such as `git rev-parse --show-toplevel`, so a later move breaks none;
- `docsLinkIsResearchData` in `internal/lymcwd/docslink_test.go` accepts the new depth;
- code comments, docs and `README.md` naming a moved path;
- relative links inside the reports.

A grep for every old path afterwards finds none outside git history.
