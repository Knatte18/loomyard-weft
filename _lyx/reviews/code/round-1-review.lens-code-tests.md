```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens — per behaviour the diff adds, the test that pins it, or none.

**Per-behaviour list**

- The diff adds no Go or production code. It changes only `docs/research/codex-review-bench-2026-10-25/` (`report.md`, `data/*`, `data/harness/*.sh`, `data/reviews/**`). No test file is added or changed.
- Harness scripts (`build.sh`, `run.sh`, `grid.sh`, `dump.sh`): no test pins them. They are one-off research tooling that writes under `.scratch/`, and no build or test reads them. The `code-tests` lens does not require a test for them. Nothing to flag.
- `data/match.tsv` and `data/usage.txt`: no test pins them. They are measurement records, so the report's counts are not behaviour a test would cover. The rubric's "Test economy" exempts static prose and records of this kind.
- `report.md`: no test pins it.
- Test-economy checks:
  - New tests duplicating an existing test: none, because no test is added.
  - Per-helper tests already covered by a public-surface test: none.
  - Review-fix tests added for a finding that was not a coverage gap: none.
  - New assertions on a stencil's static prose: none.
  - A behaviour the diff adds with no coverage: none. The diff adds no executable product behaviour, and the harness scripts are research scratch tooling outside the product.

**Findings:** none.

**Tool calls:** none made. The inherited context already held the diff stat and the harness scripts.
