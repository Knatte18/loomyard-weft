```yaml
lens: interactions
outcome: reported
```

**Pairs checked (interactions lens)**

- D1 × D2 (PR findings copy rides `pr-findings`' own commit and the status commit) — held: `pr-findings`' dirty refusal reads the code worktree where `_lym` is ignored, so an uncommitted kept file never trips it.
- D1 × D3 (ask-record copies written by another process between status commits) — held: status commit / teardown's `_lym` commit pick them up.
- D1 × D3 (status commit stages the kept dir whole while an atomic write's temp file sits in it) — broken (F1).
- D1 × D4 (copy written after fixer session, before Finalize's next locked pass) — held: the landing fixer's dirty gate reads the code worktree, where `_lym` is ignored.
- D1 × D5 (row written at Finalize end, after its records push) — held: next transition's status commit or teardown's commit carries it.
- D1 × D6 (kept dir outside reviews root vs the segment walk and extract) — held.
- D2 × D3 (PR-Review findings and PR-Review parent ask's report in sibling subdirs, numbered independently) — held.
- D2 × Constraints ("new records reach origin only through … the background pusher") — broken (F3).
- D3 × D5 (ask copy written after the index row by a late answer/expiry) — held: the row holds paths only, and no new ask is made after Finalize.
- D4 × D5 (fixer copies listed in the row) — held: Finalize's fixer runs before the landing, so "any later fixer" names nothing, but nothing breaks.
- D4 × its acceptance criterion (two attempts in one row → two reports) — broken (F2).
- D5 × its acceptance criterion (resumed Finalize → byte-identical row) and D5 × D6 (list prints the row's landing commit) — broken (F4).
- D5 × D6 ordering (tag with index row first, then tip time; diverged teardown's second tag) — held.
- D6 × D7 (list reads tags kept forever) — held.
- D2/D3/D4 × criterion "a kept file written after the last records commit is in the archive tag" — held on the remove path (`commitPendingRecords` stages structural `_lym`).
- D1 × criteria (staged by status commit; not a dirty path outside run records) — held given the pathspec and `PathsOutsideRunRecords` changes D1 names.

**Findings**

F1 — MEDIUM, design. Location: `decision-record.md` D3 (vs D1).
D3 claims its atomic writes through `internal/state` mean "a concurrent status commit never stages a half-written file", but D1 has the status commit stage the kept directory whole (`git add -- &lt;dir&gt;`), and `fsx` creates the atomic write's temp file as `.tmp-*` in the target's own directory. A commit racing a write stages a stray temp file (its removal committed later), and that churn can be what a tag holds. Fix: D3 states where the temp file lives, or D1 excludes temp names from the pathspec.
Evidence: `internal/fsx/fsx.go:57` (`os.CreateTemp(dir, ".tmp-")`); `internal/loomcli/wiring.go:99-101` (directories staged whole).

F2 — BLOCKING, design. Location: `decision-record.md` D4 and its acceptance criterion.
D4 copies the report once "after each landing fixer session ends", but one session spends several turns: a failing gate with a turn left charges the turn and re-prompts inside the same seat run, and every turn writes the one `reportPath`. The criterion "a landing fixer that runs two attempts in one row, the first failing, leaves two tracked reports named by attempt" therefore cannot hold — the first turn's report is overwritten before the session ends, so D4's purpose (every attempt kept) is missed. Fix: copy per turn inside the fixer gate's check, named by the charged attempt, or restate decision and criterion per session.
Evidence: `internal/landingshed/fixerrun.go:105`, `:143` (`Attempts: turnsLeft`), `:327-334`.

F3 — LOW, consistency. Location: `decision-record.md` Constraints vs D2.
The constraint says new records reach origin only through the run's records commits and the background pusher, but D2 puts its copy in the commit `lym loom pr-findings` makes, whose committer pushes synchronously through `pairteardown.PushRun`, not the background pusher. Fix: name the pull-request record's synchronous push as the route D2's copy takes.
Evidence: `internal/loomcli/landingdeps.go:527-545`; `PATTERN-run-fsm` background ("the pull-request record … keep their own synchronous push").

F4 — BLOCKING, design. Location: `decision-record.md` D5 (resume clause, "byte-identical row" criterion), consumed by D6.
D5 claims every field comes from durable records, but the landing commit is read live as the parent pair's `HeadSHA()` in `recordLanding` and stored only in the prime's scratch via `shedrun.RecordLandedCommit` (not tracked). A Finalize resumed past its merge reads the parent head as it stands then, possibly a later run's landing, so the resumed row names another commit and committer time; the byte-identical criterion fails, and D6 lists and extracts the wrong commit. Fix: D5 names a durable source for the landing commit (e.g. a tracked record written at the landing, or the commit found by its message/merge parent).
Evidence: `internal/landingshed/finalize.go:738-753`, `:416-419`; `internal/loomcli/landingdeps.go:249-256`.

Tool calls:
- `git check-ignore` / `git status --ignored` on `_lym` and `.lym` in the code worktree — to learn whether code-side dirty checks see kept-dir writes (they do not: `_lym` is ignored and a symlink).
- `verifyGate.dirtyPaths` in `internal/landingshed/verifygate.go` — to learn which tree the landing fixer's dirty gate reads (the code worktree).
