# Every process a run spawns has one owner that ends them all

`run-process-ownership` · Task · bug, enhancement, batten, burler, fabric, loom, shed, shuttle, webster · running · Worktree-Create

## Goal

Every process a run spawns has one owner, and the verbs that end, pause or tear down a run reach all of them.
On 2026-10-20 a paused run's driver chain and parked pushes outlived their pairs by 19 hours, and `lyx fabric remove` hung on them until the operator killed eleven processes by hand.
Operator view (2026-10-20): the many independent, long-lived processes that each drive or lock shared state are a major weakness of lyx.
Operator decision (2026-10-20): the notes below are fixed together in this one task.
Operator decision (2026-10-22): the `batten-abandon` note moves to task `batten-cancel-verb`, which owns the one verb that ends a batten run for good and calls this task's owner record to end a live run's processes; the note `review-waits-background-shells` folds in here, since its wait is the same background-work detection as `pause-background-work`.

## Note `orphan-run-processes`

Imported from [issue #760](https://github.com/Knatte18/loomyard/issues/760).

Cleaning up a second machine after two runs landed elsewhere took over ten minutes of operator time, where fabric operations should take 10–15 seconds.

### Setup

- Runs `hub-workspaces` and `implementer-seam` were started on machine A, adopted on machine B (`lyx fabric add` + `lyx batten run --window`, see #741), then paused on B with `lyx batten pause` and `lyx tender pause` the next day.
- Both runs then finished and landed from machine A.
- On B the operator asked for the two pairs to be removed with `lyx fabric remove <slug>` (no `--remote`) and the prime pulled.

### What was seen on B

1. `lyx fabric remove hub-workspaces` waited about two minutes, then failed: `pair driver is busy: the loop is live in pair "hub-workspaces" … or end it with lyx reed down in the pair`.
2. After `lyx reed down` in each pair, `remove` still hung.
3. The process table held eleven orphaned lyx processes, about 19 hours old, all from the paused runs:
   - per run, the driver chain `lyx loom drive` → `lyx loom step --until-stop --loop-detached=…` → `lyx loom step` (one `[lyx] <defunct>` child each);
   - `lyx fabric --weft-path <prime-weft> --pause-slug=hub-workspaces push`;
   - `lyx board --board-path <_board> sync` and `lyx fabric --warp-path <pair> --weft-path <pair-weft> --teardown-slug hub-workspaces --pause-slug=hub-workspaces push`, with a `lyx loom step` beside them;
   - the same `board sync` + teardown push for `implementer-seam`, spawned by the first `remove` attempt, which survived when the calling shell was stopped.
4. Every push was parked on the tender pause.
   Lifting the pause would have pushed stale records (`running · Plan-Bouncer` status, `Run-Shed -> paused` commits) over the landed state on Bolt and the weft remote.
5. After `kill` of those PIDs, `remove` finished at once.
6. Separately, Bolt (`_board`) held one unpushed local `board sync` commit and the prime weft three unpushed `batten: Run-Shed -> paused/running` commits, both diverged from origin; `lyx fabric pull` reported `weft pull failed, continuing to warp` (`pull --ff-only` exit 128) and no lyx verb could settle it.
   The operator ran `git reset --hard origin/main` in `_board` and `git reset --hard origin/main-weft` in the prime weft.

### Expected

- `tender pause` on a run either ends its driver loop and queued pushes, or `fabric remove` ends them itself, instead of leaving processes parked indefinitely.
- `fabric remove` recognises a pair whose run is paused, or whose board entry is done or landed on the parent, and does not wait on its driver; at most it names the PIDs it will end.
- A queued push under a pause is dropped, not replayed, once its pair is removed.
- A diverged local Bolt or prime weft whose local commits are only stale status/sync commits is reconciled to origin by `lyx fabric pull` (or a named verb), instead of a warning and a manual `git reset --hard`.
- `remove` should not push an archive tag of stale records from a machine that is not the run's owner (#741).

### Host

Ubuntu 24.04.5, git 2.43, tmux 3.4, lyx at `a1f435dc` (Ubuntu 24.04 support landed).

### Issue comment, 2026-10-20

### Operator note: the process model is the weakness, not only `remove`

The operator rates this a major weakness of lyx: one run leaves many independent, long-lived processes that each drive or lock shared state, with no single owner that can end them together.

What one paused run left behind here:

- a driver chain of three processes (`loom drive` → `loom step --until-stop` → `loom step`), plus zombie children;
- parked network work, each its own process holding a pending write: weft `fabric push`, `board sync`, teardown `fabric push`;
- children of an interrupted CLI call (`fabric remove`) that outlived their parent.

Together they:

- block other verbs, which detect a "live loop" and wait on it (`fabric remove` waits up to two minutes and still fails);
- hold stale writes that replay over newer remote state the moment a pause lifts;
- survive the strand teardown (`reed down`), the pause and the calling shell, so nothing in lyx sees or ends them; only `ps` and `kill` did.

#### What should hold

- Every process a run spawns belongs to one owner record (run or pair) with its PIDs or a process group, so `tender status` lists them and one verb ends them all.
- `tender pause` stops work instead of parking processes indefinitely; a parked push is a record to replay, not a live process.
- `reed down`, `fabric remove` and `batten` teardown end the pair's whole process group.
- A child of an interrupted CLI call dies with its parent (process group or `Pdeathsig`), rather than continuing with side effects.
- `tender status` reports orphaned lyx processes whose pair or run no longer exists, as it already does for processes on a replaced binary.


## Note `pause-background-work`

### Seen (2026-10-14)

`lyx hibernate request` with no trigger paused the hub at once, but its confirmation never passed: after the grace, `lyx tender status` still read `ly:overview-seats:webster` and `ly:overview-seats:driver` as `busy` with cause `background`, and the orch as `busy` with cause `turn` while it answered the operator.

- The pause interrupts a session's running turn, but work the session started in the background keeps going: Webster's implementer forks, and the driver's background wait loop.
- A session with background work reads busy, so confirmation withholds until its ten-minute bound and the request fails.
- The operator cancelled the request and hibernated by hand.

### Idea

- A hub pause also stops, or lets finish and then holds, each session's background work: forks, background shells, Monitors; or confirmation treats a session whose only background work is a known-safe wait (the driver's loop) as idle.
- What a paused fork should do (stop and be re-spawned on resume, or finish its batch and write its report) needs a design choice, since killing an implementer fork mid-batch leaves a half-done batch for Webster's recovery.
- The orch's own turn should not withhold confirmation when the orch itself requested the hibernate and ends its turn.

### Operator view (2026-10-15, not a decision)

- Claude Code can pause a fork: the fork is picked in the terminal's fork/agent picker and stopped there, and the master is then told the fork stopped. A hub pause would have to reach each fork that way.
- Forks are not interrupted as a rule: Webster forks can live long (one ran 45 minutes on 2026-10-14), and cutting one off mid-batch is out of the question as a default.
- But that 45-minute fork held 300k+ tokens of context: its batch was too long, and interrupting and restarting it with a fresh context might have been cheaper.
  This points at a separate idea: a context bound on a Webster fork that ends it and restarts the rest of the batch fresh, which also gives a pause a clean place to stop.


## Note `review-waits-background-shells`

Imported from [issue #796](https://github.com/Knatte18/loomyard/issues/796).

Seen in the quarry hub (Knatte18/quarry, run csharp-alphabet), 2026-10-21.

Plan-Burler-Review finished in about 7.5 minutes and wrote _lyx/reviews/plan/round-1-review.md with verdict APPROVED. The reviewer agent then started two background shell commands (find /). The run waited on them: lyx loom status showed 'waiting: background shells running 21m'. The operator stopped both tasks in the pane ('Stop Task … stopped') and the find processes were gone, yet the run stayed on Plan-Burler-Review with no counter moving.

Expected: a reviewer's background shells started after its verdict file is written never hold the row, or the row stops waiting once the shells have ended; a wait that long surfaces as a stop to the parent rather than a silent hang.
A separate issue covers why the reviewer searched for PATTERN files at all.



