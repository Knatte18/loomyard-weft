```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the Go and script logic this diff changes. I ran each case by hand against the diff.

**List (function or branch, case run, verdict)**

- `docsLinkIsResearchData` (`internal/lymcwd/docslink_test.go`), `docs/research/topic/x/data/prompts/step.md`: `parts[3:len-1]` is `[x data prompts]`, so it returns true. Correct.
- `docsLinkIsResearchData`, `docs/research/topic/series/x/data/output.md`: the slice is `[series x data]`, so true. Correct.
- `docsLinkIsResearchData`, `docs/research/topic/x/report.md`: the slice is `[x]`, so false and the report is scanned. Correct.
- `docsLinkIsResearchData`, `docs/research/data/x.md`: length 4 fails `len > 4`, so false. Correct. The subtest asserts this file is scanned.
- `docsLinkIsResearchData`, `docs/other/data/x.md`: `parts[1]` is not `research`, so false. Correct.
- `docsLinkIsResearchData`, boundary `docs/research/topic/data/x.md`: length 5, slice `[data]`, so true. Correct under the "data/ at any depth below the topic" contract; the subtest doesn't cover this depth.
- `docsLinkIsResearchData`, a file literally named `data`: `len-1` excludes the last element, so it is not treated as a directory. Correct.
- `docsLinkIsResearchData`, slice bounds: `len > 4` guarantees `3 <= len-1`, so there is no panic.
- Rewritten subtest, expected order: `docs/other/data/x.md`, `docs/research/data/x.md`, `docs/research/topic/x/report.md` is already ascending. Consistent with how the old `want` was ordered.
- Subtest behaviour change: the old `docs/research/x/sub/data/x.md` case was scanned (`parts[3] != "data"`) and is now skipped. This matches the new contract and is not a defect.
- `vocabScanAllowlisted`, two new exact-path entries: `docs/logging-inventory.md` and `docs/reference/weaving-glossary.md` match only by `rel == e.path`, because neither ends in `/`. They are scoped to exactly those files. Correct.
- Root lookup `git -C "$HERE" rev-parse --show-toplevel` (all changed scripts):
  - Inside the repo or a worktree it returns that checkout's root, the same directory `../../..` gave. Correct.
  - In `set -euo pipefail` scripts, a failure outside a repo aborts with git's error. Correct.
  - The `plan-writer-audit` and `batch-budget` scripts hold `REPO=...` alongside `|| exit 1` guards. Outside a repo an empty `REPO` would at worst make `OUT` resolve to `/.scratch/...` and fail at `mkdir`. The old code assumed a repo too, so this is no regression.
- `review-turn-audit-2026-11-01/run-all.sh`, `SCRATCH` unset: `rev-parse` runs and the default is `<root>/.scratch/review-turn-audit`. Correct.
- `review-turn-audit-2026-11-01/run-all.sh`, `SCRATCH` set (the staged call): no git call is made. `audit=$here/../turn-audit-2026-10-28` resolves to the staged sibling copy. Correct.
- `turn-savings-measure-2026-11-04/run-all.sh`, staging block: `cp -r "$audit" "$review" "$stage/"` puts both siblings under `$stage`. `slugs.txt` is written into `$stage/review-turn-audit-2026-11-01/`, and the staged `run-all.sh` runs with `SCRATCH=$stage/scratch`. The other staged scripts read no repo root, per the grep of `rev-parse` and `$root` uses. Correct.
- `batching-effect-audit-2026-11-01/run-all.sh` and `writer-lookups-measure-2026-11-04/run-all.sh`: `audit=$here/../turn-audit-2026-10-28`. The audit is in the same `turns/` folder, so it resolves. Correct.
- `turn-savings-measure-2026-11-04/run-all.sh`, `review=$here/../review-turn-audit-2026-11-01`: same `turns/` folder. Correct.
- `batching-effect-audit-2026-11-01/task-size.sh`: `hub=$(cd "$root/.." && pwd)` and `weft=$root/../loomyard-weft` keep the old meaning because `root` is the same directory. Correct.
- `plan-writer-audit-2026-10-24/q1-table.sh`, `RUNCOST=$REPO/docs/research/tokens/token-audit-2026-10-22/runcost.tsv`: the file exists at that path. Correct.
- Codex harness `HARNESS=`, `BLOCK=`, `DATA=` and `HARNESS_DOCS=` lines: every rewritten path matches a moved location. Correct.

**Findings**

None under this lens.

Outside my lens (affected-set): `docs/research/twill/twill-discussion-bench/deviations.md:1` and `:5` still cite `twill-discussion-bench.md`, which is now `report.md` in that folder.

**Tool calls:** none. The manifest context already held everything my procedure needed.
