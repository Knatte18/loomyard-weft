---
verdict: APPROVED
---

## Scope check against the board entry

The decision record covers every Scope point of `glyph-audit-tool`:

- One repeatable tool under `tools/`, like `tools/tokencount/tokencount.sh`, built from both audits' scripts: D1 (`tools/forkaudit/forkaudit.sh`), with the Notes naming the 2026-10-15 scripts it ports.
- Input as a date range or a list of runs, with fork discovery from the 2026-10-15 `find`: D2.
- Arm classification from the prompt and, where present, the `webster: kickstart pack` trace line: D3.
- Every listed per-fork and per-arm metric (pack opened and when, span share before the first edit and in the first work turn, first-turn parallelism, calls before the first edit, exploration calls per card, reads, greps, quarry lookups, turns, output tokens, cache reads, edits to files with no span): D4 and D5.
- Both optional sections, pack correctness against the batch's start tree and plan glyph use: D7 and D6.
- A markdown report with medians and pooled per card and sample sizes per arm: D5.
- Shell and jq only, no Python, following PATTERN and its scans: D1, Constraints, and D8's Go test files, which the Scope section explains are how `lyx gate test` reaches the tool.
- The two `docs/research/` directories left unchanged: Scope Out and an acceptance criterion.

The Out list adds nothing the entry asks for: dropping the 2026-10-13 date-based `base_on`/`mid` split fits the entry's purpose of comparing layouts by prompt, and arm `none` still covers pack-off forks.
Per-role tokens and phase timing stay with `tools/tokencount` and note `report-timing`, so the run does not duplicate that note.

The Constraints line that a task adding a pack layout must add its marker row binds later tasks such as `kickstart-per-card`.
It contradicts nothing in that entry, which keeps the layout behind a `webster.yaml` switch and has the trace line record the layout, and it does no work of that entry.
The Open risk on `kickstart-per-card`'s card-start span delivery is that entry's unbuilt format, and the record leaves it to that task rather than designing it here.

## Overlaps with runs in flight

`lyx loom peers` lists no parent commits.

- `refusal-route-config-empty` (row 12): changes begin-batch's refusal class in `internal/websterengine/beginbatch.go`, the file that logs the trace line D3 reads.
  Its decision record does not touch the `webster: kickstart pack` trace line or its `pack_layout` field, so the behaviours do not overlap and the designs do not conflict.
  If they were counted as overlapping, the peer is further along (row 12 against own row 3) and lands first.
- `run-pane-timing` (row 12): status pane and shuttle gate changes; no overlap with `tools/forkaudit`.

Shared bookkeeping: D8 adds a row to `cmd/lyx/testdata/test-budget.yaml`; no peer lists that file among its changed paths.
