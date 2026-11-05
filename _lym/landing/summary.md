# docs/research is restructured into topic folders with an index

`docs/research` now holds one folder per topic, and `docs/research/README.md` lists every study by topic with a line on what it measured.
Nothing is deleted: every file moves with `git mv`, so `git log --follow` keeps each study's history.

## What changes

- **Topic folders.** The topics are `burler`, `codex`, `creel`, `drift`, `glyph`, `loom`, `platform`, `quarry`, `reed`, `speed`, `tokens`, `turns`, `twill` and `webster`.
  A near-duplicate series gets its own folder inside its topic: `codex/astra`, `codex/v2`, `codex/review`, `codex/engine-trial`, `creel/audit`, `drift/csharp-trial`, `burler/lens-fan`, `tokens/model-downgrade`, `loom/plan-split`, `twill/plan-bench` and `webster/batch-budget`.
  The `glyph-audit-*` and `twill-*` series are their topics.
- **One layout per study.** A study with scripts or data is a folder with a `report.md`:
  - A study whose report was its folder's `README.md` gets that file renamed to `report.md`.
    These are the batch-budget trials, `batten-start`, `glyph-audit-2026-10-12`, `parallel-batches`, `plan-split-bench`, `plan-writer-audit` and `token-audit`.
  - `twill-discussion-bench.md` becomes `twill/twill-discussion-bench/report.md`, beside the inputs folder it cites.
  - `twill-plan-bench.md` and its two Sonnet follow-ups each become a folder in `twill/plan-bench/` that holds its own token tables.
  - The replay files in `plan-split-replay-2026-11-02/` join their report, `plan-split-tune-2026-11-02.md`, as `loom/plan-split/plan-split-tune-2026-11-02/`.
  - A lone report stays a single file.
- **Design notes leave `docs/research`.** `reed-proposal.md`, `reed-exploration.md`, `logging-inventory.md` and `quarry-holistic-fix-log.md` move to `docs/`, and `weaving-glossary.md` moves to `docs/reference/`.
  `docs/overview.md` lists them, and its `research/` line now describes the measurement studies.
- **References follow the move.** These references now name the new paths:
  - the code comments in `internal/batcher`, `internal/lenses`, `internal/shuttlecli`, `claudeengine` and `codexengine`;
  - the measurements table in `README.md`, which also points at the new index;
  - the two benchmark reports;
  - the docs line of `crucible/review-prompt-template.md`, which now points at the index instead of a `docs/research/<module>-*.md` pattern;
  - the paths and relative links inside the reports and the harness scripts.
- **Scripts find the root by git.** The research scripts that found the repository root by `../../..` now use `git -C <script dir> rev-parse --show-toplevel`, so a later move breaks none.
  The turn-savings script stages the review turn audit without the old `docs/research` depth.
  The review turn audit's `run-all.sh` looks the root up only for its default work directory, so the staged copy still runs with `SCRATCH` set.
- **Guards.** In `internal/lymcwd/docslink_test.go`, `docsLinkIsResearchData` now skips a study's `data/` folder at any depth under a topic, and its test row covers a series-depth study.
  The weft/warp vocabulary scan in `internal/lymcwd/enforcement_test.go` now allowlists `docs/logging-inventory.md` and `docs/reference/weaving-glossary.md`.
  Both name those tokens: one as the log directory names of its day, the other as the weaving terms it defines.
  Under `docs/research/` they were exempt.

## Design decisions

- Study folder and file names stay as they were, so the names that data tables, reports and board notes cite keep matching; only their parent folders change.
- `reed-hooks-exploration.md` and `psmux-tui-behavior.md` stay as research under `reed/`, since the entry names only `reed-exploration.md` among the reed files as a design note.
- Recorded data keeps its old paths. These records are:
  - the tool-call tables (`calls.tsv` of the turn, batching-effect and turn-savings audits);
  - one captured review finding in `concrete-review-findings-2026-11-03/data/findings.tsv`;
  - one logged `cp` command in `glyph-audit-2026-10-15/COMMANDS.md`, which names another checkout's absolute path.

  They record what sessions read at the time, so rewriting them would falsify the records.
  A grep for the old paths finds only these.
- Adding the two vocabulary allowlist entries keeps the notes' text unchanged.

## To check by hand

- Open `docs/research/README.md` on GitHub and follow a few links into the series folders.
- Optionally run one moved script, for example `docs/research/webster/batch-budget/batch-budget-trial-2026-11-01/compare.sh`, to see that it finds the repository root from its new depth.
