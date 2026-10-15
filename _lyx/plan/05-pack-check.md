# Card 5 — pack-check

**Create:**
- `tools/forkaudit/pack-entries.jq`
- `tools/forkaudit/touched-files.jq`
- `plan:tools/forkaudit#TestForkauditPackCheck` -> `func TestForkauditPackCheck(t *testing.T)`

**Edit:**
- `tools/forkaudit/forkaudit.sh`
- `tools/forkaudit/report.jq`
- `plan:tools/forkaudit#TestForkauditRefusals`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/gitkit#CommitFile`
- `docs/research/glyph-audit-2026-10-15/pack-entries.jq`
- `docs/research/glyph-audit-2026-10-15/touched-files.jq`
- `docs/research/glyph-audit-2026-10-15/check-pack.sh`
- `docs/research/glyph-audit-2026-10-15/missed-files.sh`

**Intent:**
Add the optional pack-correctness section behind `--pack-check` (decision D7).

`forkaudit.sh`:
- New flags `--pack-check` (no value) and `--code <dir>` (default the repo root); usage and header comment grow with them.
  Under `--pack-check` a missing `--code` directory exits 1 naming `--code`, and a missing `--records` directory exits 1 naming `--records`, checked in that order, `--code` before `--records`, both before the work directory is touched.
  Under `--pack-check` the report header names the resolved `--code` path.
- `--pack-check` runs card 4's batch resolution for every fork, not only those with a trace file.
- `pack-entries.jq` (ported, the hard-coded path prefix dropped) emits each pack entry of a fork: from the `<batch>-pack.md` Read result for a pack-file fork, from the prompt's `## Kick-start pack` section for an inline one.
- Each span entry is classed as `check-pack.sh` classed it, with `git -C <code> cat-file -e <startSha>` and `git -C <code> show <startSha>:<file>` and the same awk: `correct`, `correct-decl`, `stale` with the line shift, `wrong`, `nofile`, `notree` (batch unresolved or `startSha` absent from the code repo); a span in a non-`.go` file gets the signature-text check alone and, failing it, `unchecked` instead of the declaration and `wrong` steps.
  Rows go to `pack-check.tsv` in the work directory.
- `touched-files.jq` (ported, with paths made relative by stripping the fork's worktree prefix `<hub>/<run>/` instead of a hard-coded one) lists files a fork edited, wrote or read; paths under the run-record root or the logs root the script passes with `--arg`, and absolute paths outside the worktree, are skipped; each file the pack gave no span for is a missed file with `existed` `yes` or `no` from `git cat-file -e <startSha>:<file>`, or `unknown` when the batch is unresolved.
  Rows go to `missed-files.tsv`.

`report.jq` (or the script, appending a rendered section), only under `--pack-check`: a `Pack correctness at batch start` section with, per arm, the pooled count of each class and a per-fork row of class counts, and the missed-file counts split by existed-at-start, plus the missed-file list (agent id, file, edits, writes, reads, existed).

Tests:
- `TestForkauditPackCheck` places `fork-file`, `fork-file-noedit` and `fork-raf` in run `alpha` and `fork-file` again in run `beta`.
  It builds a code repo with `gitkit.CommitFile` whose `pkg/a.go`, `pkg/extra.go` and `docs/b.md` content puts each of `fork-file`'s span entries in its intended class (card 2's fixture layout) and `fork-raf`'s inline spans in `correct`, and leaves `pkg/gone.go` and `pkg/new.go` absent.
  It builds a records repo with `recordsRepo` whose `alpha` batches for `fork-file` and `fork-raf` carry that commit as `startSha`, whose `beta` batch for `fork-file` carries a well-formed SHA absent from the code repo, and which holds no batch at `fork-file-noedit`'s start.
  It asserts the section's per-fork rows:
  - `alpha`'s `fork-file` counts one each of `correct`, `correct-decl`, `stale` (with its shift), `wrong`, `nofile` and `unchecked`;
  - `fork-raf`'s inline spans are classed `correct`, so the inline entry source is read;
  - `fork-file-noedit`'s spans are all `notree` (batch unresolved), and `beta`'s `fork-file` spans are all `notree` (`startSha` absent);
  - the missed-file list holds `alpha`'s `pkg/extra.go` with existed `yes`, `pkg/new.go` with existed `no` and `fork-file-noedit`'s `pkg/other.go` with existed `unknown`, and holds neither the plan card under the run-record directory nor the absolute path outside the worktree that `fork-file` read.
- `TestForkauditRefusals` gains two rows: `--pack-check` with `--code` naming a missing directory and `--records` naming an existing empty `t.TempDir()`, so only `--code` is missing, exits 1 naming `--code`, and `--pack-check` with `--records` naming a missing directory exits 1 naming `--records`, each leaving `--work` and `--out`'s directory unchanged.

`cmd/lyx/testdata/test-budget.yaml`: `tools/forkaudit` row to 4.

**ImpactSummary:** confined to `tools/forkaudit`: two flags and an optional section; two rows join the existing refusal table and the budget row rises by one.

**Verify:**
lyx gate test --tags integration ./tools/forkaudit
lyx gate test ./cmd/lyx -- -run TestTestBudget_PackagesStayWithinBudget
