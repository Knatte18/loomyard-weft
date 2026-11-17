# Card 14 — weaver-archive-and-reset

**Create:**
- `internal/weaverengine#ResetDeps`
- `internal/weaverengine#ResetResult`
- `internal/weaverengine#Reset`
- `internal/weaverengine#archiveRunInPlace`
- `internal/weaverengine#ArchiveRunRecord`
- `internal/weaverengine_test#TestReset`
- `internal/weaverengine_test#TestArchiveRunRecord`

**Edit:**
- `internal/weaverengine#Run`
- `internal/weaverengine_test#TestRun`

**Extends:**
- `internal/weaverengine/doc.go`

**Uses:**
- `internal/websterengine#ArchiveRunRecord`
- `internal/shuttleengine#Runner.ProbeGated`
- `internal/planparser#ParsePlanFrom`

**Intent:** `lym weaver reset`'s engine and the PR rework archive of Weaver's run directory (D2, D9, D13).

- `ResetDeps` carries the geometry and told seams: `Runner StrandRunner`, `Reed shuttleengine.ReedOps`, `StateHistory` (card 6's, whose versions carry their commit), `ReadPlanAt func(rev, name string) ([]byte, bool, error)` (`fabricengine.AnchoredFileAt` over the plan directory), `ResetCode func(sha string, ownPaths []string) error` (fabric's `ResetPairCode`), `RecordPending func() (bool, error)` (whether `_lym/weaver` or the plan directory holds uncommitted changes), `Commit func(msg string) error` (fabric's commit of `_lym/weaver` and the plan directory) and `Now`.
- `Reset` refuses, changing nothing, in this order:
  1. while Weaver's run lock is held (`RunActive`), naming waiting for the row to stop or `lym loom pause --after` the implementer row;
  2. over an absent state file: while the reset-pending marker exists and `RecordPending` reports an earlier reset's restore and archive left uncommitted, it commits them, removes the marker and returns, so a reset whose commit failed is finished by running it again; a marker with nothing pending is removed, since its reset's commit landed; otherwise it refuses, naming that no Weaver run has begun and so nothing needs a reset, and commits nothing;
  3. over a corrupt state file, it reads the start commit and plan commit from the newest committed version that parses; with none it refuses naming removing the corrupt state file and moving the task branch and the weft's plan directory back by hand with git, to where each stood before the run's first commit on it;
     and while reed lists a live strand of `BatchStrandRole` or `verify-fixer`, naming `lym reed remove <guid>`;
  4. over a state with no begun batch, naming that no start commit is recorded, so nothing needs a reset;
  5. while a merge is in progress in the task worktree, naming finishing or aborting it;
  6. while a tracked path the run did not leave is uncommitted, naming committing it on the task branch or restoring it with `git checkout -- <path>`, then re-running the reset; the run's own paths are the union of its attempts' recorded `Uncommitted` paths, which the reset discards;
  7. it probes, by recorded output file through `ProbeGated`, the strand of every attempt and fixer state records, whatever its outcome;
     a probe error refuses naming the error and shuttle's way forward;
     a found run refuses naming `lym weaver reset --stop-strand` and re-stepping the row so it attaches, under `lym loom pause --after` the implementer row.
     With `stopStrands` it stops every found run through the handle's `Stop` and goes on; a stop that fails refuses naming the strand's guid and `lym reed remove <guid>`, with nothing else changed. Without the flag it stops none.
  A probe that finds none has already removed, as `ProbeGated` does, an earlier run's strand whose record is terminal or whose agent is gone.
- Then it writes the reset-pending marker, `reset-pending` in `.lym/weaver/`, before it moves, restores or archives anything, and removes it once the commit lands.
  `Run`'s first batch begin removes any marker, so a marker left by a process that died between the commit and the removal never outlives the next run.
  The bound on step 2's commit: it commits only while a reset's marker stands, never plan edits under no reset, such as a Webster run's or a plan under review.
  The one remaining window is a reset that died after its commit landed and before it removed the marker, followed by unrelated edits and a reset call before any Weaver begin.
- Then it moves the task branch to the run's start commit (the first batch's recorded start) through `ResetCode` with the run's own paths, restores the plan directory to its content at `PlanWeftCommit` (the plan parsed at that commit with `planparser.ParsePlanFrom` over `ReadPlanAt`; each plan file written back, and each plan file absent there removed), archives the run record in place (`archiveRunInPlace`: the state file, the reports directory, `journal.md`, `summary.md` and `outcome.yaml` renamed under UTC-stamped names inside `_lym/weaver/`, which no reader reads), clears `.lym/weaver/` but its lock files and the reset-pending marker, and commits the plan restore and the archived record together through `Commit`.
  A commit failure is returned naming the re-run of `lym weaver reset`, which step 2 finishes.
  Webster's run directory is untouched.
- `ArchiveRunRecord` moves every entry of Weaver's run directory into `dest`, refusing while the run lock is held and holding the state-mutation lease across the moves, as `websterengine.ArchiveRunRecord` does; an absent run directory is a no-op.

`TestReset` is tier0, over fake seams: the lock refusal; an absent state with nothing pending refuses; an absent state with uncommitted plan edits and no marker refuses and commits nothing; an absent state with the marker and a pending restore commits it and removes the marker; an absent state with the marker and nothing pending removes the marker and commits nothing; a state with no begun batch refuses; a merge in progress refuses; a foreign uncommitted path refuses naming `git checkout`, and an attempt's own uncommitted path is passed to `ResetCode`;
a found live strand refuses and stops nothing; a probe error refuses naming it; a probe that removes a terminal attempt's strand finds none and resets; a run whose spawn returned with no guid recorded is found by its output file; `--stop-strand` stops each found run and resets, and a failed stop refuses naming `lym reed remove`;
the reset moves the branch to the start, restores the plan's pending cards to their text at the run's start, renames each record file under a stamp, clears `.lym/weaver/` but its locks, leaves `_lym/webster/` as it was, commits once and leaves no marker; a commit failure names the re-run and leaves the marker;
a corrupt state resets from the last committed version, refuses while a `weaver` or a `verify-fixer` strand is live, and refuses naming the manual repair with no version.
`TestRun` gains a row: a run's first batch begin removes a stale reset-pending marker.
`TestArchiveRunRecord` is tier0: entries move into `dest`; a held lock refuses; an absent directory moves nothing.

**ImpactSummary:** Only adds to the new weaverengine package's doc; no existing symbol changes.
