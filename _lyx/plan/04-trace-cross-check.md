# Card 4 — trace-cross-check

**Create:**
- `plan:tools/forkaudit#recordsRepo` -> `func recordsRepo(t *testing.T, runs []recordsRun) string`
- `plan:tools/forkaudit#recordsRun` -> `type recordsRun struct`
- `plan:tools/forkaudit#batchFixture` -> `type batchFixture struct`
- `plan:tools/forkaudit#TestForkauditTraceCrossCheck` -> `func TestForkauditTraceCrossCheck(t *testing.T)`

**Edit:**
- `tools/forkaudit/forkaudit.sh`
- `tools/forkaudit/extract.jq`
- `tools/forkaudit/report.jq`
- `plan:tools/forkaudit#runForkaudit`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/gitkit#Git`
- `internal/gitkit#CommitFileOnBranch`
- `docs/research/glyph-audit-2026-10-15/resolve-batches.sh`
- `internal/websterengine/beginbatch.go`

**Intent:**
Resolve each fork's batch from the records repo and cross-check its prompt arm against the begin-batch `webster: kickstart pack` trace line (decisions D3's cross-check and D7's batch resolution).

`forkaudit.sh`:
- New flag `--records <dir>`, default the repo root's sibling named the repo root plus the records suffix from the variable block; its usage and header comment grow with it.
  Without `--pack-check` and `--plan` (cards 5 and 6) a missing records directory only disables the cross-check, and the header says `No cross-check: records repo <path> not found.`
- Batch resolution, per fork whose worktree `<hub>/<run>` has a trace file under the logs directory of the variable block: for each second 1 to 6 seconds before the fork's first transcript line, `git -C <records> log <refs> -S<UTC second> --format=%H -- <state path>` over only that fork's run refs, given as `--glob` patterns — the local and `origin/` records branch (run name plus the records suffix) and the tags `archive/<run>/*` — then `git show <commit>:<state path>` and the batch whose `spawnedAt` equals that second, giving batch number, cards, `startSha` and the commit.
  A pickaxe hit whose state holds no batch with that `spawnedAt` resolves nothing for that second, and the loop goes on to the next.
  Since `--glob` implies a trailing `/*` on a pattern without a glob character, the branch patterns bracket one letter of the suffix, built from the variable block, so they match the one branch exactly; run names hold no glob character (card 1's check).
  A run with none of those refs resolves nothing, so a concurrent run's batch spawned in the same window never resolves.
  Results go to `batches.tsv` in the work directory and join the record.
  Git use stays read-only: `log` and `show` only.
- Trace join: the trace lines are the `msg="webster: kickstart pack"` lines of every trace file in the fork's worktree logs directory whose `batch=` equals the resolved batch; their `time=` (local offset) is converted with `date` to UTC, and the last line before the fork's first transcript line counts; with no such line before it, the fork has no cross-check.
  Its `pack_layout=` value is the trace value.

`extract.jq` (or a post-pass in `report.jq` reading `batches.tsv` and trace values) applies D3's rules to the record:
- trace `file` agrees with arm `file`, trace `inline` with `raf` or `inline`, trace `none` with `none`; any other pairing is a disagreement and the arm stays the prompt's.
- On an `unknown` fork, a trace value matching `^[a-z0-9_-]+$` that is none of the marker table's arm names nor `unknown` (so not `file`, `raf`, `inline`, `none` or `unknown`) makes the arm `trace:<value>`; any other value leaves the fork `unknown` and is recorded as an unrecognized trace value.
- No trace file, an unresolved batch, no trace line before the fork's start or an absent records repo leaves the fork without a cross-check.

`report.jq`: the samples table gains per arm the count of forks cross-checked; under it the disagreement list (agent id, run, arm, trace value) and, once, the unrecognized trace values; the per-fork table gains batch and cross-check columns; `trace:<value>` arms sort after the marker-table arms.

Tests:
- `runForkaudit` appends `--records` naming a path under a fresh `t.TempDir()` that does not exist, unless the arguments already carry `--records`, so the rows of cards 1 and 2 never read the live records repo beside the worktree and stay host-independent.
- `recordsRun` holds a run name, the ref kind its state is committed under (`local` branch, `origin` remote-tracking branch, or `archive` tag), its `batchFixture` values (batch number, cards, `startSha`, `spawnedAt`), an optional extra state key whose value is a UTC second, and plan card files (path under the plan dir to content), which card 6 commits and cards 4 and 5 leave nil.
- `recordsRepo` builds a records git repo in `t.TempDir()` with `gitkit.Git`: an initial empty commit on `main`, then per run the branch created from it, the state file committed at the variable block's path with `gitkit.CommitFileOnBranch`, and for the `origin` kind the commit moved to `refs/remotes/origin/<run>` plus the records suffix and the local branch deleted, for the `archive` kind tagged `archive/<run>/final` and the branch deleted.
- `TestForkauditTraceCrossCheck` is table-driven; each row places forks with `placeForks`, writes trace files under `<hub>/<run>/` and builds the records repo, then asserts the per-fork arm and cross-check cells and the samples section:
  - `raf` fork with trace `inline` agrees; `file` fork with trace `file` agrees; `none` fork with trace `none` agrees;
  - `file` fork with trace `inline` is listed as a disagreement and keeps arm `file`; `file` fork with trace `perhand` is a disagreement and keeps arm `file`;
  - `unknown` fork with trace `perhand` gets arm `trace:perhand`; `unknown` forks with trace `raf`, `unknown` and `a|b` stay `unknown` and are listed as unrecognized trace values;
  - two trace lines for the fork's batch, one before and one after the fork's start, use the one before; only a line after the start gives no cross-check;
  - `fork-raf` placed in two runs, `alpha` and `beta`, whose batches share a `spawnedAt` but carry different batch numbers and trace layouts, each resolves to its own run's batch;
  - the batch committed under the `origin` kind and under the `archive` kind each resolves;
  - a fork with a trace file but no matching `spawnedAt` gets no cross-check; one whose state holds the second only in the extra key, with no batch, gets no cross-check;
  - a fork whose batch the records repo holds but whose worktree has no trace file gets no cross-check, and the samples section counts it as not cross-checked;
  - `--records` naming a missing directory gives the header's no-cross-check line, exit 0.

`cmd/lyx/testdata/test-budget.yaml`: `tools/forkaudit` row to 3.

**ImpactSummary:** confined to `tools/forkaudit`: one new flag, a records-repo read path and report columns; `runForkaudit` gains a default flag its callers do not see, and the budget row rises by one.

**Verify:**
lyx gate test --tags integration ./tools/forkaudit
lyx gate test ./cmd/lyx -- -run TestTestBudget_PackagesStayWithinBudget
