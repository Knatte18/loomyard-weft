# Card 6 — plan-glyph-use

**Create:**
- `tools/forkaudit/plan-writer.jq`
- `tools/forkaudit/testdata/plan-writer-main.jsonl`
- `tools/forkaudit/testdata/plan-writer-minor.jsonl`
- `tools/forkaudit/testdata/plan-writer-tie.jsonl`
- `tools/forkaudit#TestForkauditPlanSection`

**Edit:**
- `tools/forkaudit/forkaudit.sh`
- `tools/forkaudit/report.jq`
- `tools/forkaudit#forkFixture`
- `tools/forkaudit#placeForks`
- `tools/forkaudit#recordsRepo`
- `tools/forkaudit#TestForkauditReport`
- `tools/forkaudit#TestForkauditRefusals`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `docs/research/glyph-audit-2026-10-15/plan-refs.sh`
- `docs/research/glyph-audit-2026-10-15/plan-stats.jq`
- `docs/research/glyph-audit-2026-10-15/plan-writer.sh`
- `docs/research/glyph-audit-2026-10-15/plan-writer.jq`

**Intent:**
Add the optional plan glyph-use section behind `--plan` (decision D6).

`forkaudit.sh`:
- New flag `--plan` (no value); usage and header comment grow with it.
  Under `--plan` a missing `--records` directory exits 1 naming `--records`; a missing `--code` directory is no failure, since `--plan` reads no code tree.
- `--plan` runs card 4's batch resolution for every fork.
- Per run with forks in the report, the weft commit is the newest commit on that run's refs (card 4's `--glob` set) that touches the state path of the variable block, found with `git log -1 --format=%H`.
  The plan cards are the files `git ls-tree -r --name-only <commit> -- <plan dir>` lists, `00-overview` excluded; each is read with `git show` and its references under `Edit`, `Create`, `Move`, `Delete`, `Rename` and `Uses` are classed `glyph`, `plan:` handle, bare file or bare directory by `plan-refs.sh`'s awk, ported; a bullet under any other label is skipped.
- The plan writer of a run is, among the top-level session transcripts (`<project dir>/*.jsonl`, never `subagents/`) that write a plan card (`_lyx/plan/NN-*.md`, spelled from the variable block), the one with the most such Writes, ties broken by the lower session id; `plan-writer.jq` (ported) counts its quarry calls, greps and reads.
- A run whose weft commit or plan-writer session is not found gets one line saying which, and counts in no table.

`report.jq` (or the script), only under `--plan`: a `Plan glyph use` section with, per arm, the medians per card of each reference kind, and the plan writers' quarry calls against greps and reads (pooled and median per writer), followed by the not-found lines.
A run whose forks span several arms counts under each of those arms.

Fixtures: `plan-writer-main.jsonl` is a top-level session that Writes two plan cards and makes known counts of `lyx quarry` calls, greps and reads; `plan-writer-tie.jsonl` also Writes two plan cards, under a session id that sorts after main's; `plan-writer-minor.jsonl` Writes one card.

Tests:
- `forkFixture` gains a top-level flag, and `placeForks` places a flagged fixture as `<projects>/<encoded hub>-<run>/<session>.jsonl` instead of under `subagents/`.
- `recordsRepo` commits each `recordsRun`'s plan card files under the plan dir with its state file, in the same commit.
- `TestForkauditPlanSection` places, all with `--plan` and `--code` naming a missing directory:
  - run `alpha`: `fork-file` and `fork-raf`, and the three plan-writer sessions top-level; the records repo holds alpha's state and two plan cards whose references cover every kind (a glyph, a `plan:` handle, a bare file and a bare directory) plus a bullet under a `**Verify:**` label;
  - run `alpha-two`: `fork-inline`, its state in the records repo, no top-level plan-writer session and a copy of `plan-writer-main.jsonl` placed under `subagents/`;
  - run `gamma`: `fork-none`, with no refs in the records repo.

  It exits 0 and asserts:
  - the per-card medians of each kind for arm `file`, and the same values for arm `raf`, since alpha's forks span both arms;
  - the `**Verify:**` bullet counted in no kind;
  - the plan writer is `plan-writer-main`, over the tied `plan-writer-tie`, with its quarry, grep and read counts;
  - `alpha-two`'s one line naming the missing plan-writer session, the subagent copy not counting;
  - `gamma`'s one line naming the missing weft commit.
- `TestForkauditReport`'s `--since` row gains assertions that the report holds neither the `Pack correctness at batch start` nor the `Plan glyph use` heading, since neither flag is given.
- `TestForkauditRefusals` gains a row: `--plan` with `--records` naming a missing directory exits 1 naming `--records`.

`cmd/lyx/testdata/test-budget.yaml`: `tools/forkaudit` row to 5.

**ImpactSummary:** confined to `tools/forkaudit`: one flag and an optional section; the placement and records helpers gain a top-level session and plan cards their earlier callers do not use, and the budget row rises by one.

**Verify:**
lyx gate test --tags integration ./tools/forkaudit
lyx gate test ./cmd/lyx -- -run TestTestBudget_PackagesStayWithinBudget
