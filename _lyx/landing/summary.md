# Fast batten start, local-only pairs, one Command producer

A batten start no longer waits on GitHub between rows, fabric can create pairs that never touch origin, and batten's flag-gated side-action rows become rows of one generic `Command` producer.

## What changes

- **Detached status pushes.**
  The status seams of batten, loom and darn, and the records push of `lyx batten cleanup`, keep their synchronous commit and spawn fabric's detached push child instead of pushing inline.
  One child is spawned per committed transition, never per no-op transition: the seams' `Commit` dependency now reports whether a commit landed.
  Batten's seam and cleanup push the prime's records side only; loom's seam pushes the pair's records and code sides, and in the prime the records side only.
  A new fabric spawn entry point applies that prime rule, so no caller names a records path.
- **Push coalescing and teardown.**
  Detached children try-lock the push lock and leave a per-side pending flag when another push holds it, so concurrent children coalesce instead of queueing (`internal/fabricengine/pendingflags.go`).
  A per-slug teardown lock orders `Remove` against the pair's pushes (`internal/fabricengine/teardownlock.go`).
- **Local-only pairs.**
  `lyx fabric add --local` creates a pair with no origin probe and no push, and marks both branches local in their repos' git config.
  Every fabric push primitive skips a marked branch and reports it skipped; `lyx fabric remove` and `lyx fabric cleanup` of such a pair run no origin step.
  A local pair gets no `_portals` link and no `_launchers` entry.
- **`Command` producer.**
  `shedadapters` gains one `Command` producer, registered as the `Command` engine in `internal/shedrecipe`.
  It replaces the `OpenWindow`, `SeedIDE` and `OpenIDE` engines, their `Env` seams and `SwitchedProducer`.
  Batten's rows keep their names, so resume is unaffected, and read `batten.open_window`, `batten.seed_ide` and `batten.open_ide` as switches and the matching `*_command` keys as commands.
  The default commands call new verbs: `lyx ide seed`, `lyx ide open` and `lyx reed window`.
- **Measurement.**
  `docs/research/batten-start-2026-10-18/` records the per-row account before and after.
  It found no per-transition overhead beyond the commit and the row's own action, so no further production code was cut.

Closes no issue: the board entry lists none.

## Design decisions to review

- **Local mark.**
  The mark is a per-branch git config key written only by `add --local`.
  A pair counts as local when either surviving branch carries it, so a hand-set key makes an ordinary pair local: its remove archives nothing, drops pending records, and its pushes publish nothing.
  `remove` reports on its result that it took the local path and which branch carried the mark.
- **Local remove drops records.**
  A local pair's records exist only on its weft branch, so its remove discards them without `--force`, recording the dropped weft tip's sha.
  The warp branch keeps today's gate and is kept with `CodeBranchKeptReason` when its work is unlanded.
- **`add --local` bounds.**
  It refuses when either branch already exists unmarked, locally or as an `origin` remote-tracking ref.
  A plain `lyx fabric add` that adopts a marked branch clears the mark.
- **Command trust.**
  A command runs whatever the template or the board default layer says, with the operator's rights.
  The executable-key lookup reads only the template and the board default layer, so no task branch or board entry can set a command; set one with `lyx config set --layer default` from the prime.
  A non-zero exit, a timeout or a Windows `%` refusal leaves the row Done and logs a warning; an unreadable setting or a missing executable-key lookup stops the run at the row.
  Placeholders are exactly `{lyx}`, `{slug}` and `{task_dir}`, each quoted for the shell that runs the command; `verifyrun` gained literal-argument quoting for this.
- **Push failures.**
  A seam no longer sees a push failure; the detached child logs it at Warn under `fabricengine.HubLogsDir`, and the next transition's spawn carries the commits.

## Check by hand

- Take a live batten start with Open-Window and Seed-IDE on, and add the numbers to the research report; the after-numbers for those two rows were not measured live.
- Watch the children's logs for `(failed)` refusals from the added detached pushes to the shared records repository.
- Pending-flag protocol (`internal/fabricengine/pendingflags.go`): a review left open that it has no bound on how long a pending flag can persist or be re-spawned.
- `PATTERN-recipe-setting-lookup`: a review found the pattern does not yet tell the executable-key lookup that `Command` rows need; check that the pattern text matches the code.
- Respawned code-side push flag: a review found it lacks the weft path in `internal/fabricengine`; check the respawn pushes the right side of a pair.
- Discussion-time design findings left open: the detached child's working directory inside the task worktree, the `Command` trust bound being stated only in the decision rather than in the producer's docs, and the prime's push holding the pair's teardown lock.
- Windows paths were not exercised: `cmd` quoting and the `%` refusal for `{task_dir}` and `{lyx}`.
