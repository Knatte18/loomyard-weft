---
verdict: APPROVED
prior:
  - id: F1
    status: resolved
    answer: D4 keeps a watch map of its own keyed by slug that recordOwed never touches, D1 names d.listedRuns, and an acceptance bullet pins one landed line across several ticks before the clear.
  - id: F2
    status: resolved
    answer: D3 copies the land_notice reading into the watch's own memory while the run is ahead of Land and skips only a landed reading whose copy is on; a cancelled reading is never skipped, with acceptance bullets for both.
  - id: F3
    status: resolved
    answer: D1 makes a never-read child part absent and D3 counts an absent child part as no stop, with an acceptance bullet for a run before Run-Shed.
  - id: F4
    status: resolved
    answer: The Goal now names only a cancel that keeps its records, and D3 states that a cancel removing its records is reported by the cancel's own notice, tried once.
  - id: F5
    status: resolved
    answer: D2's stops rule is defined over D3's whole-reading predicate, and acceptance bullets cover blocked to failed and a pause lifting over a blocked child.
  - id: F6
    status: resolved
    answer: D1 states that the watch runs only on ticks whose scheduling steps run and that an anchor move, adopt hold or catch-up delays its lines.
  - id: F7
    status: resolved
    answer: D2 states that a run whose run_watch never read is not judged and keeps no memory until a read succeeds, with an acceptance bullet.
  - id: F8
    status: resolved
    answer: D1's rationale now uses a batten block outside Run-Shed, which derives as running or landing since Halted reads only the child.
  - id: F9
    status: resolved
    answer: The fixer's different fix holds. tryAcquireSlot stamps Started per slot (internal/gateslot/pool.go:322), so D6 defines a hold as one PID with all its slots and keys it by PID and earliest Started, which yields one notice per multi-slot hold.
  - id: F10
    status: resolved
    answer: D6 states that a failed Pool.Holders read skips the check for that tick with one Warn per failure episode, and the acceptance bullet covers it.
  - id: F11
    status: resolved
    answer: D6 refuses a negative gate_hold_factor and a gate_hold_min below one, names the flood bound, and an acceptance bullet pins the refusal.
  - id: F12
    status: resolved
    answer: D6's rationale pins the Go half in the runtime source (nanotime1 in src/runtime/sys_linux_amd64.s calling clock_gettime with CLOCK_MONOTONIC) beside clock_gettime(2).
  - id: F13
    status: resolved
    answer: The tolerant-records constraint now names only the dobby's state record, and D6 states that the holder record is YAML decoded without KnownFields (internal/gateslot/pool.go:360-361).
---

# Discussion review — round 2

Every round-1 finding is resolved, and this round raises no new BLOCKING finding on a changed hunk, so the verdict is APPROVED.

The focus entries were each driven to a verdict:

- **Watch memory and owed lines.** The watch map lives beside `LandNotices` in the state record. `recordOwed` prunes only `s.Runs` (`internal/dobbyengine/passes.go:333-337`).
  The watch reads `d.listedRuns`, which keeps a done run that `d.runs` drops (`passes.go:181-188`).
  On the tick batten reads done, `landNotices` decides and drops first. The watch then posts, or owes, the landed line. After that the clear (`loop.go:719`, `:729`) holds the run while a line is owed and drops the entry when it clears the run.
  Each line goes once, is retried at probe spacing, and is dropped after three failures or an hour, as D5 decides.
- **Overlap with the landing notice.**
  - Landed with `land_notice` on: the landing notice posts at Worktree-Teardown, and the watch skips the landed reading on its copied reading.
  - Landed with `land_notice` off: the watch line alone.
  - Cancelled with `land_notice` on or off: the watch line. A cancel at Worktree-Teardown also gets the landing notice's line, since `PastLand` holds there (`internal/battenrecipe/land.go:27-29`).
  - The derived state reads landed only once batten reads done (`internal/runfsm/state.go:71-76`), which comes on or after the landing notice's decision.
- **Cancel visibility.** The Goal, D3 and the acceptance criteria agree that a cancel that removes its records is reported by the cancel's own notice alone. One gap is below the incremental bar: such a cancel can still get a `cancelling` watch line. It is listed below.
- **Stop predicate.** Each reading has one stated answer:
  - no child status: absent, so no stop;
  - a batten pause lifting over a blocked child: one line;
  - blocked to failed: one line;
  - a failed first `run_watch` read: not judged;
  - a tick without scheduling: no watch step, as D1 states.
- **Gate-hold notice.** Dedup is per PID and earliest start. A failed `Pool.Holders` read skips the tick with one Warn per episode. The ranges are a factor of 0 or more and a fallback of at least one minute. Both halves of the clock fact are pinned.
  One gap is below the incremental bar: holds that are legitimate can outlast the bound. It is listed below.

## Below the incremental bar

These are real defects the lens forks found and I confirmed.
None is BLOCKING, and an incremental round admits only BLOCKING new findings, so they are not in `findings`.
The parent may want them carried into Plan or a later Discussion change.

- **MEDIUM, design. D6's bound and hold start count time the tier timeout does not bound** (walkthrough F-w1, bound-breaker F3).
  - The earliest `Started` includes a multi-slot acquire's wait for later slots (`internal/gateslot/pool.go:270-298`).
  - The shared build and `retryTier` run inside the same hold, each under its own timeout (`internal/gatecli/build.go:28-43`).
  - So a healthy gate can pass 1.5 × the tier's `timeout_min`.
  - A fix would measure the bound from the latest `Started` and size it for the build and one retry, or state this as an accepted early-notice bound.
- **MEDIUM, design. D1's episode stamp misses a second Stuck at the same row** (bound-breaker F1).
  - `appendOrFold` folds a repeated budget-exempt Stuck into the last history entry, raising `Repeats` without appending (`internal/shedengine/run.go:698-720`).
  - So the history length does not move, and the acceptance bullet on a second block at the same row fails for that case.
  - A fix would add the newest entry's `Repeats` or `LastAt` to the stamp.
- **MEDIUM, design. The watch reads runs owned by another machine** (existing-paths F-A).
  - `DobbyRuns` lists such runs and only marks them (`internal/dobbyengine/owned.go:15-30`, `passes.go:141`).
  - So a non-owner dobby can post duplicate, late lines from its pulled copy of the status.
  - D1 could state that the watch skips a run marked owned elsewhere.
- **LOW, consistency. A cancel posts a `cancelling` line** (interactions F-A, walkthrough F-w2, bound-breaker F2).
  - D3 counts cancelling as a stop, and a cancel writes its intent before it removes anything (`internal/runfsm/state.go:80-82`).
  - So a removing cancel can get a `cancelling` line, against D3's "the watch posts no line for it".
  - A keeping cancel gets two lines.
- **LOW, design. A failed gate-hold post has no stated rule** (interactions F-D).
  - D6 does not say whether its key is recorded on the attempt or on success, nor which retry and drop rule a failed post follows.
- **LOW, design. D5's one-hour cap can drop a line no post was ever tried for** (interactions F-B).
  - The cap counts awake time even across an adopt hold or catch-up, during which D1 runs no watch step.
- **LOW, consistency. D7 covers only one arrival order** (interactions F-C).
  - D7 covers a batten notice arriving after the watch line.
  - Under D1's scheduling bound, the watch line can arrive after batten's notice.

## Rejected

- No-strand reporting through `requirePosted` (existing-paths F-B): how the post seam reports a missing strand is plan-level detail. `NotifyRunFrom` already returns `posted=false` with a nil error (`internal/orchcli/notify.go:73`), so D5's rule can be built as stated.
