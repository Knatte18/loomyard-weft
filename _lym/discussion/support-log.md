## Interview

Autonomous mode: no operator turns; every pick below is an auto-pick, listed in the question ledger.

### Turn 1 — exploration

- Teardown tags the records branch tip as `archive/<slug>/<sha12>` and pushes it before deleting the branch; no code prunes these tags.
- `lym report` already enumerates archive tags and reads run records from their trees.
- A sample of recent tags holds `_lym/discussion/`, `_lym/plan/`, `_lym/reviews/code/`, `_lym/landing/summary.md`, `_lym/usage/` and friction notes.
- Not tracked today: PR-Review findings (`.scratch/`, `.lym/loom/pr-findings-<n>.md`), the parent-review ask record and reports (`.lym/parent-review/`), landing fixer reports (`.lym/shed/<run>/`).
- So the entry's premise that the decision records are gone is partly wrong: they are in the tags, and the analysis did not know where to look.

### Turn 2 — design

Settled the first draft's decisions; the lens review below reshaped them into D1 to D7.

### Turn 3 — lens review

All six lenses returned a report.

- interactions, walkthrough, bound-breaker: ask-record copy can go stale when written after the lock — fixed (D3 copies inside the locked update, atomically).
- existing-paths: copies under the reviews root show as fake segments in `lym report` — fixed (D1 puts them in one kept directory outside the reviews root, with a criterion).
- existing-paths: `_lym/landing/` files sit outside the status commit and block the batten binary re-step while uncommitted — fixed (D1 adds the kept directory to the status commit pathspec and run-records set).
- existing-paths: copies written from another process may be half-staged — fixed (D3 atomic writes via `internal/state`).
- interactions, walkthrough: fix-here answers had no number source and could collide with send-backs — fixed (D2 one shared sequence, kind in name).
- walkthrough, bound-breaker: one copy at fixer end keeps only the last attempt; number source unnamed — fixed (D4 copies at each session end, attempt number from the attempt record, kill bound stated).
- walkthrough, existing-paths: resumed Finalize could stamp a new landing time — fixed (D5 uses the landing commit's committer time).
- interactions: a fixer after the index row was written is missing from the row — fixed (D5 writes the row as Finalize's last step).
- interactions: stale ask copy after the row — rejected: the row holds paths only, and D5 now runs last.
- interactions, walkthrough, bound-breaker: "newest run" undefined, and a diverged teardown leaves two tags — fixed (D6 order: index row first, then tip time; tip time shown).
- interactions: extract of an older run had no file set — fixed (D6 fixed subtrees for every run, with a criterion).
- bound-breaker: the list reads local tags only — fixed (D6 bound names it).
- bound-breaker: D7 cited a listing function as the deleter — fixed (D7 names the cleanup file instead).
- walkthrough: D1's copy should join the verb's own commit — fixed (D2).
- walkthrough: the ask record names no report paths — fixed (D3 "every ask number that has one").
- board-scope: Describe text "as merged" not addressed — fixed (D5 names the landing commit's message).
- board-scope: index row read as per-run file — rejected as a finding: an auto-pick with its rationale (Q1); no contradiction with the board.
- leanness: Goal narrative, perishable tag tally, padding scope line, restated auto-mode assumptions, duplicate plan-writer notes, evidence-only constraint, repeated teardown sentence — fixed (trimmed).
- leanness: D3 enumerated the ask record's writers — fixed (one locked update path).
- leanness: `--slug` print untested — fixed (criterion added).

## Rejected alternatives

- A shared index file on the board branch, one row appended per landing: every landing would write and push the board branch under its lock, adding contention and a new failure mode at landing, while the archive tags already list the archived runs, and an index row inside a tag marks a landed one.
- Moving the live parent-review ask record and PR findings into `_lym/`: changes locking and commit traffic of live state; a copy in the same locked update keeps the same history.
- Copying the scratch artifacts once, at Finalize: a cancelled or crashed run would lose them; copying when each is produced keeps them for every run.
- Putting the copies under the reviews root: `lym report`'s segment walk would read them as review segments.
- A retention bound on archive tags: the operator asked that records be kept, and they are small text.

## Review rounds

### Round 1

The review returned 15 findings, two blocking; every one was fixed.

- F1: D5 names the landing record in the kept directory, written at the merge and read back on a resume.
- F2, F3, F9: D4 copies each fixer turn's report in the gate check and each fix review's report at its end, named by a sequence the copy allocates.
- F4: D1's tracked `.gitignore` ignores the atomic writes' `.tmp-*` names.
- F5: D3 seeds the live ask record from the tracked copy.
- F6, F7: D6 lists partial runs as not landed and states the extract's write rules.
- F8, F11, F12: D5 writes the row before `pushRecords`, with `_lym/`-relative paths and the landing message as merged.
- F10: D2 keys copies by content hash.
- F13, F14, F15: the Constraints, D7 and the PATTERN changes were corrected; the retention rule moved to `PATTERN-pair-remote`.
- Self-check: four fork lenses reported; their findings tightened D1's writer rule, D3's seed on read, D5's record replacement on a new merge, D6's Done test and extract sibling, and D4's rationale.

## Question ledger

- Q1 (auto-pick): where does the index row live? Picked: inside each run's records, read through archive tags (D5). Why: no shared write per landing; tags already list the archived runs.
- Q2 (auto-pick): how long are records kept? Picked: forever (D7). Why: the operator's "absolutely"; no prune exists today.
- Q3 (auto-pick): are landing fixer reports in scope? Picked: yes (D4). Why: the entry asks that findings live nowhere only in scratch, and these are the remaining fixer reports lost.
- Q4 (auto-pick): does the task cover older archived runs? Picked: yes, read-only, through a derived row and a fixed extract set in `lym report landed` (D6). Why: the existing archive tags already hold decision records and review rounds.
- Q5 (auto-pick): PR-Review is being removed by `pr-modes-no-agent`; keep its findings anyway? Picked: yes (D2), with an open risk. Why: that task has not started, and runs land with PR-Review until it does.
