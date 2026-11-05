# Restructure docs/research into topic folders with an index

`docs/research` mixed flat reports, dated folders with a `report.md`, and folders with a `README.md` and scripts, had long near-duplicate series, held design notes that are not research, and had no index.
Every study now sits in a topic folder, and `docs/research/README.md` lists each one with a line on what it measured.
Nothing is deleted; files move with their history.

## What changes

- Topic folders: `burler/`, `codex/`, `creel/`, `drift/`, `glyph/`, `loom/`, `platform/`, `quarry/`, `reed/`, `speed/`, `tokens/`, `turns/`, `twill/`, `webster/`.
- A near-duplicate series is one folder inside its topic, such as `codex/astra/`, `codex/v2/`, `creel/audit/`, `twill/plan-bench/` and `webster/batch-budget/`.
- A study with data or scripts is a folder with a `report.md`; a lone report stays a single file.
  The `README.md` of `batch-budget-trial-2026-11-01` is now its `report.md`.
- Design notes leave `docs/research`: `reed-proposal.md`, `reed-exploration.md`, `logging-inventory.md` and `quarry-holistic-fix-log.md` move to `docs/`, and `weaving-glossary.md` to `docs/reference/`.
  `docs/overview.md` links them.
- Study scripts find the repository root with `git rev-parse --show-toplevel` instead of `../../..`, so a later move breaks none of them.
- Relative links inside the reports, the root `README.md` research table, `docs/benchmarks/`, `crucible/review-prompt-template.md` and the code comments that cite a moved path are updated.
  The comments are in `internal/batcher`, `internal/lenses`, `internal/shuttlecli` and the `claudeengine` and `codexengine` packages.

## Design decisions

- `docsLinkIsResearchData` in `internal/lymcwd/docslink_test.go` now treats a `data/` directory at any depth below a topic folder as study data, so it covers studies inside series folders.
  A `data/` folder directly under `docs/research/` or directly under a topic belongs to no study and is scanned.
  The test rows pin both cases.
- `internal/lymcwd/enforcement_test.go` adds `docs/logging-inventory.md` and `docs/reference/weaving-glossary.md` to the vocabulary-scan allowlist.
  Outside `docs/research/` they would otherwise trip the scan on the record-directory names and weaving terms they spell out.
- Reports keep their text, as each records one run; only paths and links change.

## To check

- Grep the tree for each old `docs/research/<name>` path and confirm none remains outside git history.
- Run `lym gate test ./internal/lymcwd` to confirm the docs-link guard and the vocabulary scan pass.
- Run one study script from its new folder to confirm the root lookup works at depth.
